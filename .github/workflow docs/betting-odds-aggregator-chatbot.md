# Sports Betting Odds Aggregator with Responsible Gambling AI Assistant

> Agent instructions (enforced by review):
> - Implement **MVP only**. Ignore any items in `/docs/FUTURE.md` and sections labeled “Phase 2/3”.
> - When given a task, modify only files under: `api/routes`, `backend/arbitrage`, `frontend/components/Chatbot`.
> - If a change seems to require work outside the allowed areas, stop and open an issue titled `SCOPE-ESCALATION`.

## MVP Implementation Checklist (Agents MUST limit work to this)
- [ ] API: `/chat/query` endpoint (rate-limited, auth stub OK)
- [ ] Backend: `ArbitrageCalculator` minimal path (no predictive models)
- [ ] Frontend: Chatbot component wiring (no mobile)
- [ ] Responsible Gambling: static educational messages + links
- [ ] Tests: happy-path unit tests for calculator + chat route
- [ ] CI: lint/typecheck/tests pass

## Project Overview
A real-time application that aggregates sports betting odds from major betting platforms, allowing users to quickly compare odds across different bookmakers for specific games or leagues. Enhanced with an AI-powered chatbot that provides arbitrage opportunity detection while promoting responsible gambling practices and educating users about the mathematical realities of sports betting.

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
  - **NEW:** Historical performance analyzer
  - **NEW:** Betting pattern recognition
- **Technologies:**
  - Python/Node.js backend
  - Apache Kafka for event streaming
  - Redis pub/sub for real-time updates

```python
# Enhanced Arbitrage Detection with Risk Analysis
class ArbitrageCalculator:
    def __init__(self):
        self.minimum_profit_threshold = 0.042  # 4.2%
        
    def calculate_arbitrage_opportunity(self, odds_data):
        """
        Calculate arbitrage opportunities and associated risks
        
        Args:
            odds_data: List of odds from different bookmakers
            
        Returns:
            Dict containing arbitrage info and risk assessment
        """
        best_odds = self._find_best_odds_per_outcome(odds_data)
        
        # Calculate implied probabilities
        implied_total = sum(1/odd for odd in best_odds.values())
        
        if implied_total < 1:
            profit_margin = (1 - implied_total) / implied_total
            
            if profit_margin > self.minimum_profit_threshold:
                stake_distribution = self._calculate_stakes(best_odds, 1000)  # Example $1000 total
                
                return {
                    'opportunity_exists': True,
                    'profit_margin': profit_margin,
                    'expected_profit': profit_margin * 1000,
                    'stake_distribution': stake_distribution,
                    'bookmakers_involved': list(best_odds.keys()),
                    'risk_factors': self._assess_risks(best_odds, odds_data),
                    'educational_note': self._generate_educational_context(profit_margin)
                }
        
        return {
            'opportunity_exists': False,
            'market_efficiency': implied_total,
            'house_edge': (implied_total - 1) * 100,
            'educational_note': self._explain_house_edge(implied_total)
        }
    
    def _assess_risks(self, best_odds, all_odds):
        """Assess risks associated with arbitrage opportunity"""
        return {
            'account_limitations': 'Bookmakers may limit or close profitable accounts',
            'odds_movement': 'Odds can change between placing bets',
            'bet_acceptance': 'Not all bets may be accepted at listed odds',
            'market_volatility': self._calculate_odds_volatility(all_odds)
        }
    
    def _generate_educational_context(self, profit_margin):
        """Generate educational content about the rarity and challenges of arbitrage"""
        rarity_score = self._calculate_opportunity_rarity(profit_margin)
        
        return {
            'rarity_explanation': f'Arbitrage opportunities like this occur in approximately {rarity_score}% of markets',
            'practical_challenges': [
                'Requires accounts at multiple bookmakers',
                'Need significant capital for meaningful profits',
                'Time-sensitive - opportunities disappear quickly',
                'Risk of account restrictions'
            ],
            'long_term_reality': 'Most recreational bettors lose money over time despite occasional arbitrage opportunities'
        }
```

### 3. Storage Layer
- **Components:**
  - Real-time database for current odds
  - Historical database for trends and analysis
  - Caching layer
  - **NEW:** Knowledge base for AI chatbot
  - **NEW:** User interaction history
  - **NEW:** Educational content repository
- **Technologies:**
  - Redis for real-time data
  - PostgreSQL for historical data and user interactions
  - Vector database (Pinecone/Weaviate) for RAG knowledge base
  - Redis Cache

```sql
-- Enhanced PostgreSQL Schema with AI Components

-- Existing tables (events, odds) remain the same...

-- New tables for AI chatbot functionality
CREATE TABLE knowledge_base (
    kb_id SERIAL PRIMARY KEY,
    content_type VARCHAR(50), -- 'arbitrage', 'education', 'statistics'
    title VARCHAR(200),
    content TEXT,
    metadata JSONB,
    embedding_vector VECTOR(1536), -- For semantic search
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE user_interactions (
    interaction_id SERIAL PRIMARY KEY,
    user_id INTEGER,
    session_id VARCHAR(100),
    query TEXT,
    response TEXT,
    context_data JSONB,
    interaction_type VARCHAR(50), -- 'arbitrage_inquiry', 'general_question', 'education_request'
    sentiment_score DECIMAL(3,2),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE TABLE arbitrage_opportunities (
    arb_id SERIAL PRIMARY KEY,
    event_id INTEGER REFERENCES events(event_id),
    profit_margin DECIMAL(10, 6),
    stake_distribution JSONB,
    bookmakers_involved TEXT[],
    risk_assessment JSONB,
    opportunity_window INTERVAL,
    status VARCHAR(20), -- 'active', 'expired', 'executed'
    detected_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    expired_at TIMESTAMP WITH TIME ZONE
);

-- Indexes for AI functionality
CREATE INDEX idx_knowledge_base_content_type ON knowledge_base(content_type);
CREATE INDEX idx_user_interactions_type ON user_interactions(interaction_type);
CREATE INDEX idx_arbitrage_opportunities_status ON arbitrage_opportunities(status);
```

### 4. AI/RAG Chatbot Layer (NEW)
- **Components:**
  - Large Language Model integration (OpenAI GPT-4, Claude, or local models)
  - Vector database for semantic search
  - Knowledge base management system
  - Conversation context manager
  - Responsible gambling content generator
- **Technologies:**
  - Python with LangChain/LlamaIndex
  - OpenAI API or Anthropic Claude API
  - Pinecone/Weaviate for vector storage
  - Sentence Transformers for embeddings

```python
# AI Chatbot Implementation
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Pinecone
from langchain.chains import ConversationalRetrievalChain
from langchain.memory import ConversationBufferWindowMemory

class ResponsibleGamblingChatbot:
    def __init__(self, api_key, pinecone_config):
        self.llm = ChatOpenAI(
            temperature=0.3,
            model_name="gpt-4-turbo",
            openai_api_key=api_key
        )
        self.embeddings = OpenAIEmbeddings(openai_api_key=api_key)
        self.vectorstore = Pinecone.from_existing_index(
            index_name=pinecone_config['index_name'],
            embedding=self.embeddings
        )
        self.memory = ConversationBufferWindowMemory(
            memory_key="chat_history",
            return_messages=True,
            k=10  # Remember last 10 exchanges
        )
        
        # System prompt emphasizing responsible gambling
        self.system_prompt = """
        You are a responsible gambling AI assistant for a sports betting odds aggregator. 
        Your primary goals are:
        1. Provide accurate information about arbitrage opportunities when they exist
        2. Educate users about the mathematical realities of sports betting
        3. Promote responsible gambling practices
        4. Discourage problem gambling behaviors
        
        Key principles:
        - Always emphasize that the house edge ensures long-term losses for most bettors
        - Highlight the rarity and practical difficulties of profitable arbitrage
        - Provide statistical context about gambling outcomes
        - Encourage users to view gambling as entertainment, not investment
        - Suggest betting limits and self-exclusion resources when appropriate
        """
    
    def process_query(self, user_query, user_context=None):
        """Process user query with RAG and responsible gambling focus"""
        
        # Analyze query intent
        query_intent = self._analyze_query_intent(user_query)
        
        if query_intent == 'arbitrage_inquiry':
            return self._handle_arbitrage_query(user_query, user_context)
        elif query_intent == 'general_betting':
            return self._handle_general_betting_query(user_query)
        elif query_intent == 'profit_expectation':
            return self._handle_profit_expectation_query(user_query)
        else:
            return self._handle_general_query(user_query)
    
    def _handle_arbitrage_query(self, query, context):
        """Handle queries about arbitrage opportunities"""
        current_opportunities = self._get_current_arbitrage_opportunities()
        
        response_data = {
            'opportunities': current_opportunities,
            'educational_context': self._get_arbitrage_education(),
            'reality_check': self._get_arbitrage_reality_check()
        }
        
        # Generate contextual response
        prompt = f"""
        Based on the current arbitrage data: {current_opportunities}
        
        User query: {query}
        
        Provide information about any arbitrage opportunities while emphasizing:
        1. The rarity of such opportunities
        2. Practical challenges in execution
        3. The overall mathematical disadvantage in sports betting
        4. That most bettors lose money over time
        
        Be helpful but realistic and educational.
        """
        
        return self.llm.predict(prompt)
    
    def _handle_profit_expectation_query(self, query):
        """Handle queries about making money from betting"""
        educational_content = self._retrieve_educational_content('gambling_mathematics')
        
        prompt = f"""
        User is asking about making money from betting: {query}
        
        Educational context: {educational_content}
        
        Provide a response that:
        1. Explains the mathematical reality of gambling (house edge)
        2. Shows statistics on long-term gambling outcomes
        3. Explains why most people lose money
        4. Suggests viewing gambling as entertainment only
        5. Provides resources for responsible gambling
        
        Be empathetic but clear about the mathematical realities.
        """
        
        return self.llm.predict(prompt)
    
    def _get_arbitrage_reality_check(self):
        """Generate reality check content for arbitrage opportunities"""
        return {
            'success_rate': 'Less than 1% of sports bettors are profitable long-term',
            'arbitrage_frequency': 'True arbitrage opportunities occur in <0.5% of markets',
            'execution_challenges': [
                'Requires significant capital',
                'Bookmaker account limitations',
                'Odds changes during execution',
                'Maximum bet restrictions'
            ],
            'alternative_suggestion': 'Consider investing in index funds for actual returns'
        }
    
    def _retrieve_educational_content(self, topic):
        """Retrieve relevant educational content from knowledge base"""
        relevant_docs = self.vectorstore.similarity_search(
            topic, 
            k=3,
            filter={"content_type": "education"}
        )
        return [doc.page_content for doc in relevant_docs]
```

### 5. Application Layer (Enhanced)
- **Components:**
  - RESTful/GraphQL API
  - Authentication service
  - User preferences service
  - Notification service
  - **NEW:** AI Chatbot API endpoints
  - **NEW:** Responsible gambling tracking
- **Technologies:**
  - Express.js/FastAPI backend
  - JWT for authentication
  - WebSockets for real-time updates
  - Docker for containerization

```javascript
// Enhanced Express.js API Routes with AI Chatbot
const express = require('express');
const router = express.Router();
const chatbotController = require('../controllers/chatbot.controller');
const auth = require('../middleware/auth');
const rateLimiter = require('../middleware/rateLimiter');

// Existing routes remain the same...

// New AI Chatbot routes
router.post('/chat/query', 
    rateLimiter({ windowMs: 60000, max: 20 }), // 20 requests per minute
    auth, 
    chatbotController.processQuery
);

router.get('/chat/educational-content/:topic', 
    auth, 
    chatbotController.getEducationalContent
);

router.get('/arbitrage/opportunities', 
    auth, 
    chatbotController.getCurrentArbitrageOpportunities
);

router.post('/user/gambling-limits', 
    auth, 
    chatbotController.setGamblingLimits
);

router.get('/user/interaction-history', 
    auth, 
    chatbotController.getInteractionHistory
);

module.exports = router;
```

### 6. Frontend Layer (Enhanced)
- **Components:**
  - Responsive web application
  - Mobile applications
  - Real-time odds display
  - Interactive visualizations
  - **NEW:** AI Chatbot interface
  - **NEW:** Responsible gambling dashboard
- **Technologies:**
  - React/Next.js for web frontend
  - React Native for mobile apps
  - D3.js/Chart.js for visualizations
  - TailwindCSS for styling

```jsx
// AI Chatbot Component
import React, { useState, useEffect, useRef } from 'react';
import { sendChatQuery, getChatHistory } from '../api/chatbotApi';

const ResponsibleGamblingChatbot = () => {
  const [messages, setMessages] = useState([
    {
      type: 'bot',
      content: `Hi! I'm here to help you understand betting odds and arbitrage opportunities. 
      
      **Important**: Sports betting involves significant risk. The house edge ensures that most bettors lose money over time. I'll provide accurate information while helping you understand these realities.
      
      How can I help you today?`,
      timestamp: new Date()
    }
  ]);
  const [inputMessage, setInputMessage] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const messagesEndRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(scrollToBottom, [messages]);

  const handleSendMessage = async (e) => {
    e.preventDefault();
    if (!inputMessage.trim() || isLoading) return;

    const userMessage = {
      type: 'user',
      content: inputMessage,
      timestamp: new Date()
    };

    setMessages(prev => [...prev, userMessage]);
    setInputMessage('');
    setIsLoading(true);

    try {
      const response = await sendChatQuery(inputMessage);
      
      const botMessage = {
        type: 'bot',
        content: response.message,
        metadata: response.metadata, // May include arbitrage data, educational links, etc.
        timestamp: new Date()
      };

      setMessages(prev => [...prev, botMessage]);
    } catch (error) {
      console.error('Chat error:', error);
      const errorMessage = {
        type: 'bot',
        content: 'I apologize, but I encountered an error. Please try again.',
        timestamp: new Date()
      };
      setMessages(prev => [...prev, errorMessage]);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="chatbot-container h-96 flex flex-col border rounded-lg">
      <div className="chatbot-header bg-blue-600 text-white p-4 rounded-t-lg">
        <h3 className="font-semibold">Responsible Gambling Assistant</h3>
        <p className="text-sm opacity-90">Get informed about odds and arbitrage opportunities</p>
      </div>
      
      <div className="messages-container flex-1 overflow-y-auto p-4 space-y-4">
        {messages.map((message, index) => (
          <div key={index} className={`message ${message.type === 'user' ? 'user-message' : 'bot-message'}`}>
            <div className={`p-3 rounded-lg ${
              message.type === 'user' 
                ? 'bg-blue-500 text-white ml-auto max-w-xs'
                : 'bg-gray-100 text-gray-800 mr-auto max-w-md'
            }`}>
              <div className="whitespace-pre-wrap">{message.content}</div>
              
              {message.metadata && (
                <div className="mt-2 pt-2 border-t border-gray-200">
                  {message.metadata.arbitrage_opportunity && (
                    <div className="arbitrage-info text-sm">
                      <strong>Arbitrage Opportunity Detected:</strong>
                      <ul className="mt-1 space-y-1">
                        <li>Profit Margin: {(message.metadata.profit_margin * 100).toFixed(2)}%</li>
                        <li>Bookmakers: {message.metadata.bookmakers.join(', ')}</li>
                        <li className="text-amber-600 font-medium">⚠️ Remember: Execution challenges apply</li>
                      </ul>
                    </div>
                  )}
                  
                  {message.metadata.educational_links && (
                    <div className="educational-links mt-2">
                      <strong>Learn More:</strong>
                      <ul className="mt-1">
                        {message.metadata.educational_links.map((link, i) => (
                          <li key={i}>
                            <a href={link.url} className="text-blue-600 hover:underline text-sm">
                              {link.title}
                            </a>
                          </li>
                        ))}
                      </ul>
                    </div>
                  )}
                </div>
              )}
            </div>
            <div className="text-xs text-gray-500 mt-1">
              {message.timestamp.toLocaleTimeString()}
            </div>
          </div>
        ))}
        
        {isLoading && (
          <div className="bot-message">
            <div className="bg-gray-100 text-gray-800 p-3 rounded-lg mr-auto max-w-md">
              <div className="flex items-center space-x-2">
                <div className="animate-pulse">Thinking...</div>
                <div className="flex space-x-1">
                  <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce"></div>
                  <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '0.1s' }}></div>
                  <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '0.2s' }}></div>
                </div>
              </div>
            </div>
          </div>
        )}
        
        <div ref={messagesEndRef} />
      </div>
      
      <div className="chatbot-input border-t p-4">
        <form onSubmit={handleSendMessage} className="flex space-x-2">
          <input
            type="text"
            value={inputMessage}
            onChange={(e) => setInputMessage(e.target.value)}
            placeholder="Ask about arbitrage opportunities, betting odds, or gambling risks..."
            className="flex-1 border border-gray-300 rounded-lg px-3 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
            disabled={isLoading}
          />
          <button
            type="submit"
            disabled={isLoading || !inputMessage.trim()}
            className="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed"
          >
            Send
          </button>
        </form>
        
        <div className="mt-2 text-xs text-gray-500">
          This AI provides educational information about betting. Remember: most people lose money gambling.
        </div>
      </div>
    </div>
  );
};

export default ResponsibleGamblingChatbot;
```

### 7. Analytics Layer (Enhanced)
- **Components:**
  - User behavior tracking
  - Odds movement analysis
  - Predictive models
  - Business intelligence dashboards
  - **NEW:** AI interaction analytics
  - **NEW:** Responsible gambling metrics
  - **NEW:** Educational content effectiveness tracking
- **Technologies:**
  - Google Analytics/Mixpanel
  - Python (Pandas, NumPy, scikit-learn)
  - Tableau/PowerBI for visualization

## Enhanced API Endpoints Reference

### AI Chatbot Endpoints
- `POST /api/chat/query` - Send query to AI chatbot
- `GET /api/chat/educational-content/:topic` - Get educational content
- `GET /api/arbitrage/opportunities` - Get current arbitrage opportunities
- `POST /api/user/gambling-limits` - Set user gambling limits
- `GET /api/user/interaction-history` - Get chat interaction history

### Knowledge Management Endpoints  
- `POST /api/admin/knowledge-base` - Add educational content (admin)
- `PUT /api/admin/knowledge-base/:id` - Update knowledge base entry (admin)
- `GET /api/educational/statistics` - Get gambling statistics and facts

## Knowledge Base Content Strategy

### Educational Content Categories
1. **Mathematical Reality**
   - House edge explanations
   - Probability theory basics
   - Expected value calculations
   - Long-term outcome statistics

2. **Arbitrage Education**
   - What is arbitrage betting
   - Practical challenges and limitations
   - Success rate statistics
   - Account management risks

3. **Responsible Gambling**
   - Setting limits and budgets
   - Recognizing problem gambling signs
   - Resources and helplines
   - Alternative investment suggestions

4. **Market Analysis**
   - How odds are calculated
   - Market efficiency concepts
   - Factors affecting odds movement
   - Professional vs recreational betting

## Responsible Gambling Features

### Built-in Protections
- **Loss Tracking**: Monitor user losses over time
- **Time Limits**: Encourage breaks from the platform
- **Reality Checks**: Regular reminders about gambling risks
- **Educational Interventions**: Proactive educational content delivery
- **Resource Links**: Easy access to gambling addiction resources

### AI-Powered Interventions
```python
class ResponsibleGamblingMonitor:
    def __init__(self, user_id):
        self.user_id = user_id
        
    def assess_user_risk(self, interaction_history):
        """Assess user's gambling risk based on chat interactions"""
        risk_indicators = {
            'profit_focused_queries': 0,
            'ignoring_educational_content': 0,
            'unrealistic_expectations': 0,
            'frequency_of_queries': 0
        }
        
        # Analyze chat patterns for concerning behavior
        for interaction in interaction_history:
            if self._indicates_profit_obsession(interaction.query):
                risk_indicators['profit_focused_queries'] += 1
                
        risk_score = self._calculate_risk_score(risk_indicators)
        
        if risk_score > 0.7:  # High risk
            return {
                'risk_level': 'high',
                'interventions': [
                    'Show prominent loss statistics',
                    'Suggest gambling addiction resources',
                    'Limit arbitrage opportunity notifications',
                    'Increase educational content frequency'
                ]
            }
        elif risk_score > 0.4:  # Medium risk
            return {
                'risk_level': 'medium',
                'interventions': [
                    'Increase educational messaging',
                    'Show reality check statistics',
                    'Suggest setting betting limits'
                ]
            }
        else:
            return {'risk_level': 'low', 'interventions': []}
```

## Deployment Architecture (Enhanced)

### AI/ML Infrastructure
- **Vector Database**: Pinecone/Weaviate for RAG knowledge base
- **LLM API**: OpenAI GPT-4 or Anthropic Claude API
- **Model Serving**: Optional local model deployment with vLLM/Ollama
- **Embedding Service**: Sentence Transformers or OpenAI embeddings

### Monitoring and Observability
- **AI Interaction Monitoring**: Track query types, response quality, user satisfaction
- **Responsible Gambling Metrics**: Monitor intervention effectiveness
- **Educational Impact**: Track user learning and behavior change

## Development Roadmap (Updated)

### Phase 1: MVP with Basic AI (8-10 weeks)
- Core data collection from 3-5 major bookmakers
- Basic odds comparison interface
- Simple AI chatbot with educational focus
- User authentication system
- Responsive web application

### Phase 2: Enhanced AI and Arbitrage Detection (6-8 weeks)
- Advanced arbitrage opportunity detection
- RAG-powered chatbot with comprehensive knowledge base
- Mobile application development
- Personalization and favorites
- Responsible gambling monitoring

### Phase 3: Advanced AI Features (8-10 weeks)
- Predictive modeling for odds movements
- Advanced user risk assessment
- Social features with educational focus
- Professional-grade analytics dashboard
- Integration with gambling addiction resources

## Ethical Considerations and Legal Compliance

### Responsible AI Implementation
- **Transparency**: Clear disclosure of AI capabilities and limitations
- **Bias Prevention**: Regular auditing of AI responses for harmful biases
- **Educational Priority**: AI optimized for education over engagement
- **Harm Reduction**: Proactive identification and prevention of problem gambling behaviors

### Legal Compliance
- **Jurisdiction Awareness**: Comply with local gambling laws and regulations
- **Age Verification**: Robust age verification systems
- **Self-Exclusion**: Integration with national self-exclusion databases
- **Advertising Standards**: Compliance with responsible gambling advertising codes

## Getting Started (Updated)

### Prerequisites
- Node.js (v16+)
- Python (3.8+)
- Docker and Docker Compose
- PostgreSQL
- Redis
- Vector Database (Pinecone account or self-hosted Weaviate)
- OpenAI API key or Anthropic API key

### Setup Instructions
1. Clone the repository
2. Copy `.env.example` to `.env` and configure environment variables including AI API keys
3. Run `docker-compose up` to start development environment
4. Initialize vector database with educational content: `npm run seed-knowledge-base`
5. Run database migrations: `npm run migrate`
6. Start the development server: `npm run dev`

### Environment Variables (Updated)
```
# AI/ML Configuration
OPENAI_API_KEY=your_openai_api_key
PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=betting-knowledge-base
EMBEDDING_MODEL=text-embedding-ada-002

# Responsible Gambling Settings
ENABLE_RISK_MONITORING=true
MAX_QUERIES_PER_DAY=100
EDUCATIONAL_CONTENT_FREQUENCY=high
```

## Success Metrics

### User Education Metrics
- **Knowledge Retention**: Pre/post interaction knowledge tests
- **Behavior Change**: Reduced high-risk gambling indicators
- **Educational Engagement**: Time spent on educational content
- **Resource Utilization**: Usage of responsible gambling resources

### Business Metrics
- **User Retention**: Focus on educated, responsible users
- **Platform Safety**: Reduced problem gambling indicators
- **Educational Impact**: Measurable increase in gambling literacy
- **Arbitrage Discovery**: Legitimate opportunities identified vs. user expectations managed

This enhanced system provides valuable arbitrage information while prioritizing user education and responsible gambling practices, creating a more ethical and sustainable approach to sports betting odds aggregation.