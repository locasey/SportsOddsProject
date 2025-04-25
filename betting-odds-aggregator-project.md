# Sports Betting Odds Aggregator Project

## Project Overview
A real-time application that aggregates sports betting odds from major betting platforms, allowing users to quickly compare odds across different bookmakers for specific games or leagues.

## System Architecture

### 1. Data Collection Layer
- **Components:**
  - API Clients for major betting platforms
  - Web Scrapers for sites without accessible APIs
  - Rate-limiting and request caching mechanisms
  - Data normalization pipeline
- **Technologies:**
  - Python (Requests, BeautifulSoup, Selenium)
  - Node.js for API integrations
  - Redis for request caching

```python
# Example API Client Structure
class BettingAPIClient:
    def __init__(self, api_key, base_url):
        self.api_key = api_key
        self.base_url = base_url
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        })
    
    def get_odds(self, sport, event_id=None):
        endpoint = f'/odds/{sport}'
        if event_id:
            endpoint += f'/{event_id}'
        response = self.session.get(f'{self.base_url}{endpoint}')
        return self._process_response(response)
    
    def _process_response(self, response):
        # Handle rate limiting, errors, and normalize data
        pass
```

### 2. Data Processing Layer
- **Components:**
  - Odds normalization engine
  - Event matching service
  - Arbitrage opportunity calculator
  - Data enrichment service
- **Technologies:**
  - Python/Node.js backend
  - Apache Kafka for event streaming
  - Redis pub/sub for real-time updates

```python
# Odds Normalization Example
def convert_odds(value, from_format, to_format):
    """
    Convert odds between American, Decimal, and Fractional formats
    
    Args:
        value: The odds value
        from_format: Source format ('american', 'decimal', 'fractional')
        to_format: Target format ('american', 'decimal', 'fractional')
    
    Returns:
        Converted odds value
    """
    # Convert to decimal format as intermediate step
    if from_format == 'american':
        if value > 0:
            decimal = (value / 100) + 1
        else:
            decimal = (100 / abs(value)) + 1
    elif from_format == 'fractional':
        num, den = map(int, value.split('/'))
        decimal = (num / den) + 1
    else:
        decimal = value
    
    # Convert from decimal to target format
    if to_format == 'american':
        if decimal >= 2:
            return (decimal - 1) * 100
        else:
            return -100 / (decimal - 1)
    elif to_format == 'fractional':
        # Find simplest fractional representation
        fraction = Fraction(decimal - 1).limit_denominator(100)
        return f"{fraction.numerator}/{fraction.denominator}"
    else:
        return decimal
```

### 3. Storage Layer
- **Components:**
  - Real-time database for current odds
  - Historical database for trends and analysis
  - Caching layer
- **Technologies:**
  - Redis for real-time data
  - PostgreSQL/MongoDB for historical data
  - Redis Cache

```sql
-- PostgreSQL Schema Examples

-- Events table
CREATE TABLE events (
    event_id SERIAL PRIMARY KEY,
    external_id VARCHAR(100),
    source VARCHAR(50),
    sport VARCHAR(50),
    league VARCHAR(50),
    home_team VARCHAR(100),
    away_team VARCHAR(100),
    start_time TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Odds table
CREATE TABLE odds (
    odds_id SERIAL PRIMARY KEY,
    event_id INTEGER REFERENCES events(event_id),
    bookmaker VARCHAR(50),
    market_type VARCHAR(50),
    selection VARCHAR(100),
    american_odds INTEGER,
    decimal_odds DECIMAL(10, 5),
    timestamp TIMESTAMP WITH TIME ZONE,
    is_live BOOLEAN,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create indexes
CREATE INDEX idx_events_sport_league ON events(sport, league);
CREATE INDEX idx_odds_event_bookmaker ON odds(event_id, bookmaker);
```

### 4. Application Layer
- **Components:**
  - RESTful/GraphQL API
  - Authentication service
  - User preferences service
  - Notification service
- **Technologies:**
  - Express.js/FastAPI backend
  - JWT for authentication
  - WebSockets for real-time updates
  - Docker for containerization

```javascript
// Express.js API Route Example

const express = require('express');
const router = express.Router();
const oddsController = require('../controllers/odds.controller');
const auth = require('../middleware/auth');

// Public routes
router.get('/sports', oddsController.getSports);
router.get('/leagues/:sportId', oddsController.getLeagues);
router.get('/events/:leagueId', oddsController.getEvents);
router.get('/odds/:eventId', oddsController.getOdds);

// Protected routes
router.get('/favorites', auth, oddsController.getFavorites);
router.post('/favorites/:eventId', auth, oddsController.addFavorite);
router.delete('/favorites/:eventId', auth, oddsController.removeFavorite);
router.get('/alerts', auth, oddsController.getAlerts);
router.post('/alerts', auth, oddsController.createAlert);

module.exports = router;
```

### 5. Frontend Layer
- **Components:**
  - Responsive web application
  - Mobile applications
  - Real-time odds display
  - Interactive visualizations
- **Technologies:**
  - React/Next.js for web frontend
  - React Native for mobile apps
  - D3.js/Chart.js for visualizations
  - TailwindCSS for styling

```jsx
// React Component Example for Odds Comparison

import React, { useState, useEffect } from 'react';
import { fetchOddsForEvent } from '../api/oddsApi';

const OddsComparison = ({ eventId }) => {
  const [odds, setOdds] = useState([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  const [oddsFormat, setOddsFormat] = useState('american'); // 'american', 'decimal', 'fractional'
  
  useEffect(() => {
    const loadOdds = async () => {
      try {
        setIsLoading(true);
        const data = await fetchOddsForEvent(eventId);
        setOdds(data);
        setError(null);
      } catch (err) {
        setError('Failed to load odds data');
        console.error(err);
      } finally {
        setIsLoading(false);
      }
    };
    
    loadOdds();
    
    // Set up WebSocket for real-time updates
    const ws = new WebSocket(`wss://api.example.com/odds/live/${eventId}`);
    
    ws.onmessage = (event) => {
      const updatedOdds = JSON.parse(event.data);
      setOdds(current => {
        // Update only changed odds
        const updated = [...current];
        updatedOdds.forEach(newOdd => {
          const index = updated.findIndex(o => 
            o.bookmaker === newOdd.bookmaker && 
            o.market_type === newOdd.market_type &&
            o.selection === newOdd.selection
          );
          if (index >= 0) {
            updated[index] = newOdd;
          }
        });
        return updated;
      });
    };
    
    return () => {
      ws.close();
    };
  }, [eventId]);
  
  // Group odds by market type and selection
  const groupedOdds = odds.reduce((acc, odd) => {
    const key = `${odd.market_type}_${odd.selection}`;
    if (!acc[key]) {
      acc[key] = {
        market_type: odd.market_type,
        selection: odd.selection,
        bookmakers: []
      };
    }
    acc[key].bookmakers.push({
      name: odd.bookmaker,
      odds: odd[`${oddsFormat}_odds`]
    });
    return acc;
  }, {});
  
  // Highlight best odds for each selection
  Object.values(groupedOdds).forEach(group => {
    if (group.bookmakers.length > 0) {
      if (oddsFormat === 'american') {
        // For American odds, higher positive number or lower negative number is better
        group.bookmakers.sort((a, b) => {
          if (a.odds >= 0 && b.odds >= 0) return b.odds - a.odds;
          if (a.odds < 0 && b.odds < 0) return a.odds - b.odds;
          return b.odds - a.odds;
        });
      } else {
        // For decimal and fractional, higher is better
        group.bookmakers.sort((a, b) => b.odds - a.odds);
      }
      group.bookmakers[0].isBest = true;
    }
  });
  
  if (isLoading) return <div>Loading odds...</div>;
  if (error) return <div className="error">{error}</div>;
  
  return (
    <div className="odds-comparison">
      <div className="format-selector">
        <button 
          className={oddsFormat === 'american' ? 'active' : ''} 
          onClick={() => setOddsFormat('american')}
        >
          American
        </button>
        <button 
          className={oddsFormat === 'decimal' ? 'active' : ''} 
          onClick={() => setOddsFormat('decimal')}
        >
          Decimal
        </button>
        <button 
          className={oddsFormat === 'fractional' ? 'active' : ''} 
          onClick={() => setOddsFormat('fractional')}
        >
          Fractional
        </button>
      </div>
      
      {Object.values(groupedOdds).map(group => (
        <div key={`${group.market_type}_${group.selection}`} className="market-card">
          <h3>{group.market_type} - {group.selection}</h3>
          <div className="bookmakers-grid">
            {group.bookmakers.map(bm => (
              <div 
                key={bm.name} 
                className={`bookmaker ${bm.isBest ? 'best-odds' : ''}`}
              >
                <div className="bookmaker-name">{bm.name}</div>
                <div className="odds-value">{bm.odds}</div>
              </div>
            ))}
          </div>
        </div>
      ))}
    </div>
  );
};

export default OddsComparison;
```

### 6. Analytics Layer
- **Components:**
  - User behavior tracking
  - Odds movement analysis
  - Predictive models
  - Business intelligence dashboards
- **Technologies:**
  - Google Analytics/Mixpanel
  - Python (Pandas, NumPy, scikit-learn)
  - Tableau/PowerBI for visualization

## Deployment Architecture
- Containerized microservices with Docker
- Kubernetes for orchestration
- AWS/GCP cloud hosting
- CI/CD pipeline with GitHub Actions

## Development Roadmap

### Phase 1: MVP (6-8 weeks)
- Core data collection from 3-5 major bookmakers
- Basic odds comparison interface
- User authentication system
- Responsive web application

### Phase 2: Enhanced Features (4-6 weeks)
- Mobile application development
- Personalization and favorites
- Odds movement alerts
- Additional bookmakers integration

### Phase 3: Advanced Features (8-10 weeks)
- Arbitrage opportunity detection
- Advanced analytics and predictions
- Live/in-play odds tracking
- Social features (sharing, commenting)

## API Endpoints Reference

### Public Endpoints
- `GET /api/sports` - List all available sports
- `GET /api/leagues/:sportId` - List leagues for a sport
- `GET /api/events/:leagueId` - List events for a league
- `GET /api/odds/:eventId` - Get odds for a specific event

### Protected Endpoints
- `GET /api/user/favorites` - Get user's favorite events
- `POST /api/user/favorites/:eventId` - Add event to favorites
- `DELETE /api/user/favorites/:eventId` - Remove event from favorites
- `GET /api/user/alerts` - Get user's odds alerts
- `POST /api/user/alerts` - Create new odds alert
- `DELETE /api/user/alerts/:alertId` - Delete an alert

## Environment Setup and Configuration
```
# .env.example

# API Configuration
PORT=3000
NODE_ENV=development
API_VERSION=v1

# Database Configuration
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=odds_aggregator
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT Authentication
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRATION=86400

# External API Keys
FANDUEL_API_KEY=your_fanduel_api_key
DRAFTKINGS_API_KEY=your_draftkings_api_key
BETMGM_API_KEY=your_betmgm_api_key
CAESARS_API_KEY=your_caesars_api_key

# Logging
LOG_LEVEL=debug

# Web Client URL
CLIENT_URL=http://localhost:3001
```

## Project Structure (Example for Node.js Backend)
```
betting-odds-aggregator/
├── api/                  # API Server
│   ├── src/
│   │   ├── config/       # Configuration files
│   │   ├── controllers/  # Request handlers
│   │   ├── models/       # Database models
│   │   ├── routes/       # API routes
│   │   ├── services/     # Business logic
│   │   ├── utils/        # Helper functions
│   │   └── app.js        # Express application
│   ├── tests/            # Unit and integration tests
│   ├── package.json
│   └── Dockerfile
├── data-collector/       # Data collection service
│   ├── src/
│   │   ├── clients/      # API clients for bookmakers
│   │   ├── scrapers/     # Web scrapers
│   │   ├── processors/   # Data processors
│   │   └── index.js      # Main entry point
│   ├── tests/
│   ├── package.json
│   └── Dockerfile
├── web-client/           # React web application
│   ├── src/
│   │   ├── components/   # UI components
│   │   ├── pages/        # Page components
│   │   ├── hooks/        # Custom React hooks
│   │   ├── services/     # API services
│   │   ├── store/        # State management
│   │   └── App.jsx       # Main component
│   ├── public/
│   ├── package.json
│   └── Dockerfile
├── mobile-app/           # React Native mobile app
│   ├── src/
│   │   ├── components/
│   │   ├── screens/
│   │   ├── services/
│   │   └── App.jsx
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml    # Development environment setup
├── k8s/                  # Kubernetes manifests
└── README.md
```

## Getting Started

### Prerequisites
- Node.js (v16+)
- Python (3.8+)
- Docker and Docker Compose
- PostgreSQL
- Redis

### Setup Instructions
1. Clone the repository
2. Copy `.env.example` to `.env` and configure environment variables
3. Run `docker-compose up` to start development environment
4. Run database migrations: `npm run migrate`
5. Start the development server: `npm run dev`

## Considerations for Production
- Implement rate limiting for API endpoints
- Set up comprehensive monitoring and alerting
- Configure proper security headers and CORS policies
- Implement database read replicas for scaling
- Set up backup strategies for data
- Consider using a CDN for static assets
- Implement proper error handling and logging
