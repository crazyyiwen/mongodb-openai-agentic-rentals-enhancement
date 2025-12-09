# Development Guide

## Table of Contents
1. [Project Overview](#project-overview)
2. [Project Structure](#project-structure)
3. [Technology Stack](#technology-stack)
4. [Architecture](#architecture)
5. [Core Components](#core-components)
6. [API Endpoints](#api-endpoints)
7. [Database Schema](#database-schema)
8. [AI/RAG Integration](#ai-rag-integration)
9. [Development Workflow](#development-workflow)
10. [Testing](#testing)
11. [Deployment](#deployment)

---

## Project Overview

**MongoDB AI Rental Search** is a modern, AI-powered rental property search application that leverages:
- **MongoDB Atlas Vector Search** for semantic property discovery
- **OpenAI Agents SDK** for intelligent, conversational AI assistance
- **Bun runtime** for high-performance JavaScript execution
- **Elysia.js** as a fast, type-safe web framework

The application provides:
- Natural language property search using vector embeddings
- AI chat assistant with rental-specific tools
- User authentication and profile management
- Saved rentals and search history
- Real-time conversation storage
- Advanced filtering and pagination

---

## Project Structure

```
mongodb-openai-agentic-rentals-enhancement/
│
├── public/                          # Frontend static files
│   ├── index.html                   # Main HTML page
│   ├── script.js                    # Frontend JavaScript (state management, API calls, UI)
│   └── styles.css                   # CSS styling
│
├── src/                             # Backend source code
│   ├── agents/                      # AI Agent implementations
│   │   └── rental-rag-agent.js     # RAG Agent with OpenAI Agents SDK
│   │
│   ├── config/                      # Configuration files
│   │   ├── database.js             # MongoDB connection & database manager
│   │   └── vector-search.js        # Vector search index configuration
│   │
│   ├── controllers/                 # Request handlers
│   │   ├── auth.controller.js      # Authentication logic
│   │   ├── chat.controller.js      # Chat/AI assistant logic
│   │   └── rental.controller.js    # Rental CRUD operations
│   │
│   ├── middleware/                  # Express-style middleware
│   │   ├── auth.js                 # JWT authentication middleware
│   │   ├── cors.js                 # CORS configuration
│   │   ├── errorHandler.js         # Error handling middleware
│   │   └── logger.js               # Request logging middleware
│   │
│   ├── models/                      # Data models
│   │   ├── conversation.js         # Conversation/chat history model
│   │   ├── rental.js               # Rental property model
│   │   └── user.js                 # User model with auth
│   │
│   ├── routes/                      # API route definitions
│   │   ├── auth.routes.js          # Authentication routes
│   │   ├── chat.routes.js          # Chat/AI routes
│   │   └── rental.routes.js        # Rental routes
│   │
│   ├── services/                    # Business logic services
│   │   └── vector-search.service.js # Vector search & hybrid search
│   │
│   └── server.js                    # Main server entry point
│
├── .env                             # Environment variables (not in git)
├── .gitignore                       # Git ignore rules
├── package.json                     # Project dependencies
├── bun.lock                         # Bun lockfile
├── README.md                        # User-facing documentation
├── Development.md                   # This file - developer documentation
├── server.js                        # Legacy server (replaced by src/server.js)
└── seed-hf-airbnb-data.js          # Data seeding script
```

---

## Technology Stack

### Backend
- **Runtime:** Bun (fast JavaScript runtime)
- **Framework:** Elysia.js v1.0
  - High-performance, type-safe web framework
  - Built-in schema validation with Zod
  - Native TypeScript support
- **Database:** MongoDB Atlas
  - Vector Search for semantic similarity
  - Full-text search
  - Aggregation pipelines
- **AI/ML:**
  - OpenAI Agents SDK (@openai/agents)
  - GPT-4o-mini for chat
  - text-embedding-3-small for embeddings (1536 dimensions)
- **Authentication:** JWT (jsonwebtoken) + bcrypt

### Frontend
- **Vanilla JavaScript** (ES6+)
- **HTML5** + **CSS3**
- **Font Awesome** for icons
- **Marked.js** for markdown rendering
- **Highlight.js** for code syntax highlighting

### Development Tools
- **dotenv** for environment management
- **Swagger/OpenAPI** for API documentation
- **Zod** for schema validation

---

## Architecture

### High-Level Architecture

```
┌─────────────────┐
│   Frontend      │
│  (HTML/CSS/JS)  │
└────────┬────────┘
         │ HTTP/REST
         ▼
┌─────────────────┐
│  Elysia Server  │
│   (Bun Runtime) │
├─────────────────┤
│   Middleware    │
│ - CORS          │
│ - Auth (JWT)    │
│ - Logger        │
│ - Error Handler │
└────────┬────────┘
         │
    ┌────┴─────┬──────────┬──────────┐
    │          │          │          │
    ▼          ▼          ▼          ▼
┌─────┐   ┌──────┐   ┌──────┐   ┌──────┐
│Auth │   │Rental│   │Chat  │   │Search│
│ API │   │ API  │   │ API  │   │ API  │
└──┬──┘   └───┬──┘   └───┬──┘   └───┬──┘
   │          │          │          │
   └──────────┴──────────┴──────────┘
              │
              ▼
      ┌───────────────┐
      │   Models      │
      │ - User        │
      │ - Rental      │
      │ - Conversation│
      └───────┬───────┘
              │
      ┌───────┴───────┬────────────┐
      │               │            │
      ▼               ▼            ▼
┌──────────┐   ┌────────────┐  ┌────────────┐
│ MongoDB  │   │ RAG Agent  │  │Vector Search│
│  Atlas   │   │  (OpenAI)  │  │  Service   │
└──────────┘   └────────────┘  └────────────┘
```

### Data Flow

**1. User Search Request:**
```
User Input → Frontend → API (/search) → RentalController
→ RentalModel → MongoDB (traditional query) → Response
```

**2. AI Chat Request:**
```
User Message → Frontend → API (/chat) → ChatController
→ RAG Agent → OpenAI API (with tools)
    ├─ searchRentals tool → VectorSearchService → MongoDB Vector Search
    ├─ getPropertyDetails tool → VectorSearchService → MongoDB
    └─ getSavedRentals tool → UserModel → MongoDB
→ AI Response → ConversationModel (store) → Response
```

**3. Authentication Flow:**
```
User Login → Frontend → API (/auth/login) → AuthController
→ UserModel → bcrypt.compare → JWT.sign → Token Response
```

---

## Core Components

### 1. Database Configuration (`src/config/database.js`)

**Purpose:** Centralized MongoDB connection management

**Key Features:**
- Singleton pattern for database connection
- Automatic connection handling
- Graceful shutdown support
- Collection getters for type safety

```javascript
class Database {
  async connect() // Initialize MongoDB connection
  async disconnect() // Close connection
  async ping() // Health check
  getRentalsCollection() // Get rentals collection
  getDatabase() // Get database instance
}
```

### 2. Vector Search Service (`src/services/vector-search.service.js`)

**Purpose:** Semantic search using MongoDB Atlas Vector Search

**Key Features:**
- OpenAI embedding generation (text-embedding-3-small, 1536D)
- Vector search with filters
- Hybrid search (vector + traditional)
- Flexible ID handling (ObjectId, numeric, string)

**Search Filters:**
- `property_type`, `room_type` (exact match)
- `min_price`, `max_price` (range)
- `min_bedrooms`, `min_accommodates` (minimum)
- `superhost_only` (boolean)
- `location` (market/city filter)
- `country` (country filter)

**Methods:**
```javascript
generateEmbedding(text) // Create vector embedding
vectorSearch(queryText, filters, limit) // Semantic search
traditionalSearch(queryText, filters, limit) // Regex-based search
hybridSearch(queryText, filters, limit) // Combined approach
getPropertyById(propertyId) // Fetch by ID
```

### 3. RAG Agent (`src/agents/rental-rag-agent.js`)

**Purpose:** AI assistant powered by OpenAI Agents SDK

**Model:** GPT-4o-mini (note: code shows gpt-5-mini but should be gpt-4o-mini)

**Tools Available:**
1. **searchRentals** - Semantic property search
   - Natural language query processing
   - Filter extraction and application
   - Returns ranked results with metadata

2. **getPropertyDetails** - Fetch detailed property info
   - Full property data retrieval
   - Formatted response with all details

3. **getSavedRentals** - Access user's saved properties
   - Requires authentication
   - Optional full details fetching
   - Comparison support

**Agent Instructions:**
- Available markets listed (Istanbul, Montreal, Barcelona, etc.)
- Location mapping guidelines (e.g., "Manhattan" → "New York")
- Response formatting with markdown
- Context-aware responses (current property, search state)

**Key Methods:**
```javascript
chat(userMessage, conversationHistory, userId) // Process chat
streamChat(userMessage, conversationHistory) // Streaming response
handleSearchRentals({ query, filters, limit }) // Tool handler
handleGetPropertyDetails({ propertyId }) // Tool handler
handleGetSavedRentals({ includeDetails }) // Tool handler
```

### 4. Models

#### User Model (`src/models/user.js`)

**Schema:**
```javascript
{
  username: String,
  password_hash: String, // bcrypt hashed
  created_at: Date,
  updated_at: Date,
  profile: {
    preferences: {},
    search_history: Array, // Last 50 searches
    favorite_locations: Array,
    saved_rentals: Array, // { rental_id, saved_at, rental_data }
    last_login: Date
  },
  memory_stats: {
    total_conversations: Number,
    total_searches: Number,
    memory_entries: Number
  }
}
```

**Key Methods:**
```javascript
createUser(username, password)
authenticateUser(username, password)
getUserById(userId)
updateUserProfile(userId, profileUpdates)
saveRental(userId, rentalId, rentalData)
unsaveRental(userId, rentalId)
getSavedRentals(userId)
isRentalSaved(userId, rentalId)
addToSearchHistory(userId, searchQuery, filters)
incrementUserStats(userId, statType)
```

#### Rental Model (`src/models/rental.js`)

**Schema (from Hugging Face dataset):**
```javascript
{
  _id: ObjectId | Number | String, // Flexible ID format
  name: String,
  summary: String,
  description: String,
  property_type: String, // "Apartment", "House", etc.
  room_type: String, // "Entire home/apt", "Private room"
  price: Number,
  bedrooms: Number,
  bathrooms: Number,
  accommodates: Number,
  beds: Number,
  minimum_nights: Number,
  maximum_nights: Number,
  instant_bookable: Boolean,
  address: {
    street: String,
    neighbourhood: String,
    market: String, // City name
    country: String,
    country_code: String
  },
  host: {
    host_id: String,
    host_name: String,
    host_is_superhost: Boolean,
    host_picture_url: String,
    host_response_rate: Number,
    host_response_time: String
  },
  amenities: Array<String>,
  images: {
    picture_url: String,
    thumbnail_url: String
  },
  review_scores: {
    review_scores_rating: Number, // Out of 100
    review_scores_cleanliness: Number,
    review_scores_communication: Number,
    review_scores_location: Number
  },
  number_of_reviews: Number,
  text_embeddings: Array<Number> // 1536-dimensional vector
}
```

**Projections:**
- `FRONTEND_PROJECTION` - Minimal fields for listings (excludes heavy fields)
- `DETAILED_PROJECTION` - Full details for single property view
- `SEARCH_PROJECTION` - Optimized for search results

**Key Methods:**
```javascript
findMany(query, options) // List with pagination
findById(id, detailed) // Get single rental
create(rentalData) // Create new rental
updateById(id, updateData) // Update rental
deleteById(id) // Delete rental
search(searchParams, options) // Advanced search
getStats() // Analytics aggregation
```

#### Conversation Model (`src/models/conversation.js`)

**Schema:**
```javascript
{
  sessionId: String, // Unique session identifier
  userId: ObjectId | null, // null for anonymous
  isAuthenticated: Boolean,
  messages: [
    {
      id: String,
      role: "user" | "assistant",
      content: String,
      timestamp: Date,
      metadata: {
        userId: ObjectId | null,
        isAuthenticated: Boolean,
        // Additional context
      }
    }
  ],
  createdAt: Date,
  updatedAt: Date,
  metadata: {
    totalMessages: Number,
    lastActivity: Date,
    userType: "authenticated" | "anonymous"
  }
}
```

**Key Methods:**
```javascript
createConversation(sessionId, userId)
addMessage(sessionId, role, content, metadata, userId)
getConversationHistory(sessionId, limit)
updateConversationMetadata(sessionId, metadata)
deleteConversation(sessionId)
cleanupOldConversations(daysOld)
getConversationStats()
```

### 5. Controllers

#### Auth Controller (`src/controllers/auth.controller.js`)

**Endpoints:**
- `POST /auth/register` - Create new user
- `POST /auth/login` - Authenticate user
- `GET /auth/profile` - Get user profile (protected)
- `PUT /auth/profile` - Update profile (protected)
- `POST /auth/logout` - Logout user
- `GET /auth/validate` - Validate JWT token
- `GET /auth/stats` - User statistics
- `POST /auth/saved-rentals/:rentalId` - Save rental (protected)
- `DELETE /auth/saved-rentals/:rentalId` - Remove saved rental (protected)
- `GET /auth/saved-rentals` - Get saved rentals (protected)
- `GET /auth/saved-rentals/:rentalId/check` - Check if rental is saved (protected)

**JWT Configuration:**
- Secret: `process.env.JWT_SECRET`
- Expiration: `process.env.JWT_EXPIRES_IN` (default: 7d)
- Header format: `Authorization: Bearer <token>`

#### Chat Controller (`src/controllers/chat.controller.js`)

**Key Features:**
- Session-based conversation tracking
- Context-aware AI responses
- User activity tracking (authenticated users)
- Search history recording
- Metadata extraction for UI updates

**Methods:**
```javascript
handleChatMessage({ message, conversation_history, context, sessionId, userId })
handleStreamChat({ message, conversation_history, context, sessionId, userId })
getConversationHistory(sessionId, limit)
deleteConversation(sessionId)
cleanupOldConversations(daysOld)
trackUserActivity(userId, activityType, metadata)
```

**Context Enhancement:**
- Includes current search query
- Applies active filters
- Adds currently viewed property info
- Passes user authentication status

#### Rental Controller (`src/controllers/rental.controller.js`)

**Standard CRUD Operations:**
- `getAllRentals` - List with filters & pagination
- `getRentalById` - Single rental details
- `createRental` - Add new property
- `updateRental` - Modify existing property
- `deleteRental` - Remove property
- `searchRentals` - Advanced search
- `getStats` - Analytics data

### 6. Middleware

#### Authentication Middleware (`src/middleware/auth.js`)

**Methods:**
- `generateToken(userId, username)` - Create JWT
- `verifyToken(token)` - Validate JWT
- `authenticateToken(req, res, next)` - Require auth
- `optionalAuth(req, res, next)` - Optional auth
- `requireAuth(req, res, next)` - Enforce auth

#### CORS Middleware (`src/middleware/cors.js`)

**Configuration:**
- Origin: `*` (all origins)
- Methods: GET, POST, PUT, DELETE, OPTIONS
- Headers: Content-Type, Authorization
- Preflight handling for OPTIONS requests

#### Error Handler (`src/middleware/errorHandler.js`)

**Error Types:**
- `VALIDATION` - 400 Bad Request
- `NOT_FOUND` - 404 Not Found
- `PARSE` - 400 Bad Request (invalid JSON)
- `INTERNAL_SERVER_ERROR` - 500 Internal Server Error

#### Logger Middleware (`src/middleware/logger.js`)

**Logging:**
- Request logging: `[timestamp] METHOD /path`
- Response logging: `[timestamp] METHOD /path - STATUS`

---

## API Endpoints

### Authentication Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/auth/register` | No | Register new user |
| POST | `/auth/login` | No | Login user |
| POST | `/auth/logout` | No | Logout user |
| GET | `/auth/profile` | Yes | Get user profile |
| PUT | `/auth/profile` | Yes | Update user profile |
| GET | `/auth/validate` | Yes | Validate JWT token |
| GET | `/auth/stats` | No | Get user statistics |
| POST | `/auth/saved-rentals/:rentalId` | Yes | Save rental |
| DELETE | `/auth/saved-rentals/:rentalId` | Yes | Remove saved rental |
| GET | `/auth/saved-rentals` | Yes | Get saved rentals |
| GET | `/auth/saved-rentals/:rentalId/check` | Yes | Check if saved |

### Rental Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/rentals` | No | List rentals with filters |
| GET | `/rentals/:id` | No | Get rental by ID |
| POST | `/rentals` | No | Create new rental |
| PUT | `/rentals/:id` | No | Update rental |
| DELETE | `/rentals/:id` | No | Delete rental |

**Query Parameters for `/rentals`:**
- `limit` (default: 20, max: 100)
- `skip` or `page`
- `sortBy` (default: "price")
- `sortOrder` (1 for asc, -1 for desc)
- `property_type`, `room_type`, `country`
- `min_price`, `max_price`
- `min_bedrooms`, `min_bathrooms`, `min_accommodates`
- `superhost_only`, `instant_bookable`
- `min_rating`
- `location` (market/city)
- `ids` (comma-separated for AI results)

### Search Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/search` | No | Advanced search with filters |

### Chat Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/chat` | Optional | Send message to AI assistant |
| POST | `/chat/stream` | Optional | Streaming AI response |
| GET | `/chat/history/:sessionId` | No | Get conversation history |
| DELETE | `/chat/history/:sessionId` | No | Delete conversation |
| GET | `/chat/stats` | No | Conversation statistics |
| POST | `/chat/cleanup` | No | Cleanup old conversations |
| GET | `/chat/health` | No | Health check |

**Chat Request Body:**
```javascript
{
  message: String (required, max 1000 chars),
  sessionId: String | null (optional),
  conversation_history: Array<{ role, content }> (optional),
  context: {
    current_search: String (optional),
    filters: Object (optional),
    user_preferences: Object (optional),
    current_property: Object (optional)
  }
}
```

**Chat Response:**
```javascript
{
  success: Boolean,
  message: String, // AI response
  sessionId: String,
  timestamp: String,
  context: {
    tool_calls_made: Number,
    has_rental_results: Boolean,
    search_metadata: {
      search_performed: Boolean,
      search_type: String,
      search_query: String,
      search_filters: Object,
      rental_ids: Array<String> // For UI to fetch
    }
  }
}
```

### Analytics Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/stats` | No | Rental statistics |

**Stats Response:**
```javascript
{
  success: true,
  data: {
    overall: {
      total_rentals: Number,
      avg_price: Number,
      min_price: Number,
      max_price: Number,
      avg_rating: Number
    },
    byPropertyType: Array<{
      property_type: String,
      count: Number,
      avg_price: Number
    }>,
    byCountry: Array<{
      country: String,
      count: Number,
      avg_price: Number
    }>
  }
}
```

### Health Endpoints

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/health` | No | System health check |

---

## Database Schema

### Collections

#### 1. `rentals` Collection

**Indexes:**
```javascript
// Regular indexes
{ price: 1 }
{ property_type: 1 }
{ room_type: 1 }
{ accommodates: 1 }
{ bedrooms: 1 }
{ bathrooms: 1 }
{ 'address.country': 1 }
{ 'address.market': 1 }
{ 'host.host_is_superhost': 1 }
{ name: "text", summary: "text", description: "text" }

// Vector Search Index (rental_vector_search)
{
  "fields": [
    {
      "type": "vector",
      "path": "text_embeddings",
      "numDimensions": 1536,
      "similarity": "cosine"
    },
    { "type": "filter", "path": "property_type" },
    { "type": "filter", "path": "room_type" },
    { "type": "filter", "path": "address.country" },
    { "type": "filter", "path": "price" },
    { "type": "filter", "path": "bedrooms" },
    { "type": "filter", "path": "bathrooms" },
    { "type": "filter", "path": "accommodates" },
    { "type": "filter", "path": "host.host_is_superhost" }
  ]
}
```

#### 2. `users` Collection

**Indexes:**
```javascript
{ username: 1 } // Unique
{ 'profile.last_login': 1 }
```

#### 3. `conversations` Collection

**Indexes:**
```javascript
{ sessionId: 1 } // Unique
{ userId: 1 }
{ 'metadata.lastActivity': 1 }
{ updatedAt: 1 }
```

---

## AI/RAG Integration

### Vector Search Pipeline

**1. Query Embedding Generation:**
```javascript
const embedding = await openai.embeddings.create({
  model: "text-embedding-3-small",
  input: userQuery
});
```

**2. Vector Search Query:**
```javascript
db.rentals.aggregate([
  {
    $vectorSearch: {
      index: "rental_vector_search",
      path: "text_embeddings",
      queryVector: embedding, // 1536-dimensional array
      numCandidates: 100,
      limit: 10,
      filter: {
        property_type: { $eq: "Apartment" },
        price: { $gte: 100, $lte: 500 },
        bedrooms: { $gte: 2 },
        "host.host_is_superhost": { $eq: true }
      }
    }
  },
  {
    $project: {
      name: 1,
      price: 1,
      // ... other fields
      score: { $meta: "vectorSearchScore" }
    }
  }
])
```

**3. Hybrid Search Strategy:**
```javascript
// 70% weight to vector search results
// 30% weight to traditional regex search results
// Deduplicate and sort by relevance score
```

### OpenAI Agents SDK Integration

**Agent Configuration:**
```javascript
new Agent({
  name: "RentalAssistant",
  model: "gpt-4o-mini",
  instructions: "...", // Detailed system prompt
  tools: [searchRentalsTool, getPropertyDetailsTool, getSavedRentalsTool]
})
```

**Tool Definition Example:**
```javascript
tool({
  name: 'searchRentals',
  description: 'Search for rental properties...',
  parameters: z.object({
    query: z.string(),
    filters: z.object({
      property_type: z.string().nullable().optional(),
      min_price: z.number().nullable().optional(),
      // ...
    }),
    limit: z.number().default(5)
  }),
  execute: handleSearchRentals
})
```

**Agent Execution:**
```javascript
const result = await run(agent, [
  ...conversationHistory,
  user(userMessage)
]);

// Extract tool calls and metadata
const toolCalls = extractToolCallsFromResult(result);
const metadata = extractSearchMetadata(toolCalls);
```

---

## Development Workflow

### Setup

1. **Install Bun:**
```bash
curl -fsSL https://bun.sh/install | bash
```

2. **Install Dependencies:**
```bash
bun install
```

3. **Configure Environment (.env):**
```bash
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net/rental_app
OPENAI_API_KEY=sk-...
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=7d
NODE_ENV=development
PORT=3001
```

4. **Seed Database:**
```bash
node seed-hf-airbnb-data.js
```

5. **Create Vector Search Index:**
   - Navigate to MongoDB Atlas → Cluster → Search
   - Create "rental_vector_search" index using config from `src/config/vector-search.js`

### Running the Application

**Development Mode (with hot reload):**
```bash
bun run dev
# or
bun --watch src/server.js
```

**Production Mode:**
```bash
export NODE_ENV=production
bun start
```

**Access Points:**
- Frontend: http://localhost:3001
- API: http://localhost:3001/
- Swagger Docs: http://localhost:3001/swagger
- Health Check: http://localhost:3001/health

### Development Best Practices

1. **Code Style:**
   - Use ES6+ features
   - Async/await for asynchronous operations
   - Destructuring for cleaner code
   - Modular architecture with separation of concerns

2. **Error Handling:**
   - Always wrap async operations in try-catch
   - Return structured error responses
   - Log errors with context
   - Use appropriate HTTP status codes

3. **Security:**
   - Never commit `.env` files
   - Hash passwords with bcrypt (12 salt rounds)
   - Validate all user input with Zod schemas
   - Use JWT for stateless authentication
   - Sanitize MongoDB queries to prevent injection

4. **Performance:**
   - Use database projections to limit data transfer
   - Implement pagination for large result sets
   - Cache frequently accessed data
   - Use indexes for common queries

5. **Database Best Practices:**
   - Use connection pooling (MongoDB driver default)
   - Close connections gracefully on shutdown
   - Use aggregation pipelines for complex queries
   - Monitor slow queries in Atlas

---

## Testing

### Manual Testing

**1. Test Authentication:**
```bash
# Register
curl -X POST http://localhost:3001/auth/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123"}'

# Login
curl -X POST http://localhost:3001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123"}'

# Use token
curl http://localhost:3001/auth/profile \
  -H "Authorization: Bearer <TOKEN>"
```

**2. Test Rental Search:**
```bash
# Basic search
curl "http://localhost:3001/rentals?limit=5&sortBy=price"

# Advanced search
curl "http://localhost:3001/search?location=New%20York&min_price=100&max_price=500&min_bedrooms=2"
```

**3. Test AI Chat:**
```bash
curl -X POST http://localhost:3001/chat \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Find me a 2 bedroom apartment in Manhattan under $500",
    "sessionId": null,
    "context": {
      "filters": {}
    }
  }'
```

---

## Deployment

### MongoDB Atlas Setup

1. **Create Cluster:**
   - M10+ tier required for Vector Search
   - Configure network access (whitelist IPs)
   - Create database user

2. **Create Vector Search Index:**
   - Use Atlas UI or CLI
   - Index name: `rental_vector_search`
   - Path: `text_embeddings`
   - Dimensions: 1536
   - Similarity: cosine

3. **Seed Data:**
   - Run `seed-hf-airbnb-data.js`
   - Verify data integrity
   - Check index status (Active)

### Environment Variables (Production)

```bash
MONGODB_URI=mongodb+srv://...
OPENAI_API_KEY=sk-...
JWT_SECRET=<strong-random-secret>
JWT_EXPIRES_IN=7d
NODE_ENV=production
PORT=3001
```

### Monitoring

**Key Metrics to Track:**
- API response times
- Database query performance
- Vector search latency
- OpenAI API usage and costs
- Memory and CPU usage
- Error rates
- User authentication success/failure rates

---

## Common Issues & Troubleshooting

### 1. Vector Search Not Working

**Symptoms:** No results from semantic search

**Solutions:**
- Check vector search index status in Atlas (must be "Active")
- Verify `text_embeddings` field exists in documents
- Ensure index name is `rental_vector_search`
- Check OpenAI API key is valid
- Verify embeddings are 1536-dimensional

### 2. Rate Limiting from Hugging Face

**Symptoms:** 429 errors when seeding

**Solutions:**
- Already fixed with retry logic and exponential backoff
- Increase delay between requests if needed
- Use smaller batch sizes

### 3. JWT Authentication Issues

**Symptoms:** 401/403 errors

**Solutions:**
- Check `JWT_SECRET` is set
- Verify token format: `Authorization: Bearer <token>`
- Check token expiration
- Ensure user exists in database

---

## Resources

- [Elysia.js Documentation](https://elysiajs.com/)
- [MongoDB Atlas Vector Search](https://www.mongodb.com/docs/atlas/atlas-vector-search/)
- [OpenAI Agents SDK](https://github.com/openai/openai-agents)
- [Bun Documentation](https://bun.sh/docs)
- [JWT Best Practices](https://jwt.io/)

---

**Last Updated:** 2025-12-08
