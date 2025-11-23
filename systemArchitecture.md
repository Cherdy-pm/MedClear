# MedClear System Architecture

> **Version:** 1.0.0  
> **Last Updated:** November 2025  
> **Status:** Technical Design / Architecture Specification

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture Principles](#architecture-principles)
- [High-Level Architecture](#high-level-architecture)
- [System Components](#system-components)
- [Database Design](#database-design)
- [API Design](#api-design)
- [Security & Compliance](#security--compliance)
- [Infrastructure](#infrastructure)
- [Scalability Strategy](#scalability-strategy)
- [Monitoring & Observability](#monitoring--observability)
- [Disaster Recovery](#disaster-recovery)
- [Technology Decisions](#technology-decisions)

---

## 🎯 Overview

MedClear is a mobile-first medication education platform requiring high availability, HIPAA compliance, and real-time content delivery. This document outlines the technical architecture supporting the product requirements defined in [README.md](./README.md).

### System Requirements

**Functional Requirements:**
- Serve medication information to 10,000+ concurrent users
- Sub-2-second response time for medication lookups
- Support for iOS, Android, and web platforms
- Offline capability for core content
- Real-time pharmacist Q&A (async messaging)
- Multi-language support (English, Spanish)

**Non-Functional Requirements:**
- **Availability:** 99.9% uptime (SLA)
- **Security:** HIPAA-compliant, SOC 2 Type II
- **Performance:** < 2s API response time (p95)
- **Scalability:** Support 100K users by Month 12
- **Data Integrity:** Zero data loss, automatic backups
- **Compliance:** HIPAA, GDPR, FDA regulations

---

## 🏗️ Architecture Principles

### 1. **Mobile-First Design**
- Native mobile apps (React Native) for best UX
- Progressive Web App (PWA) for web access
- Offline-first architecture for core features

### 2. **Microservices Architecture**
- Loosely coupled services for independent scaling
- API Gateway for unified interface
- Service mesh for inter-service communication

### 3. **API-First Development**
- RESTful APIs for all client-server communication
- GraphQL for complex queries (future)
- Versioned APIs for backward compatibility

### 4. **Data-Driven Decision Making**
- Event tracking for all user interactions
- A/B testing infrastructure
- Real-time analytics dashboards

### 5. **Security by Design**
- Zero-trust architecture
- Encryption at rest and in transit
- Role-based access control (RBAC)
- Audit logging for all data access

### 6. **Scalability & Performance**
- Horizontal scaling for all services
- Caching layers (Redis, CDN)
- Database read replicas
- Asynchronous processing for non-critical tasks

---

## 📐 High-Level Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   iOS App    │  │ Android App  │  │   Web App    │          │
│  │ (React       │  │ (React       │  │ (React.js +  │          │
│  │  Native)     │  │  Native)     │  │  Next.js)    │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                  │                  │                   │
│         └──────────────────┴──────────────────┘                  │
│                            │                                      │
└────────────────────────────┼──────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY LAYER                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│                    ┌──────────────────┐                          │
│                    │   API Gateway    │                          │
│                    │  (Kong / AWS     │                          │
│                    │   API Gateway)   │                          │
│                    └────────┬─────────┘                          │
│                             │                                     │
│         ┌───────────────────┼───────────────────┐                │
│         │                   │                   │                │
│         ▼                   ▼                   ▼                │
│  ┌──────────┐        ┌──────────┐        ┌──────────┐           │
│  │  Auth    │        │  Rate    │        │  Load    │           │
│  │ Service  │        │ Limiting │        │ Balancer │           │
│  └──────────┘        └──────────┘        └──────────┘           │
│                                                                   │
└────────────────────────────┬──────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MICROSERVICES LAYER                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Medication  │  │     User     │  │  Pharmacist  │          │
│  │   Service    │  │   Service    │  │ Q&A Service  │          │
│  │              │  │              │  │              │          │
│  │ - Drug Info  │  │ - Profiles   │  │ - Questions  │          │
│  │ - Summaries  │  │ - Auth       │  │ - Answers    │          │
│  │ - Interactions│ │ - Preferences│  │ - Routing    │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                  │                  │                   │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐          │
│  │  Reminder    │  │   Analytics  │  │   Content    │          │
│  │   Service    │  │   Service    │  │  Management  │          │
│  │              │  │              │  │   Service    │          │
│  │ - Push       │  │ - Events     │  │ - Videos     │          │
│  │ - SMS        │  │ - Metrics    │  │ - Images     │          │
│  │ - Email      │  │ - Reports    │  │ - i18n       │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
└────────────────────────────┬──────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  PostgreSQL  │  │    Redis     │  │Elasticsearch │          │
│  │  (Primary)   │  │   (Cache)    │  │  (Search)    │          │
│  │              │  │              │  │              │          │
│  │ - Users      │  │ - Sessions   │  │ - Drug       │          │
│  │ - Medications│  │ - Freq Data  │  │   Database   │          │
│  │ - Q&A        │  │ - Job Queue  │  │   Search     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   AWS S3     │  │  RabbitMQ    │  │  CloudWatch  │          │
│  │  (Storage)   │  │  (Queue)     │  │   Logs       │          │
│  │              │  │              │  │              │          │
│  │ - Videos     │  │ - Async Jobs │  │ - App Logs   │          │
│  │ - Images     │  │ - Emails     │  │ - Metrics    │          │
│  │ - Backups    │  │ - Notifs     │  │ - Traces     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                   EXTERNAL INTEGRATIONS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Twilio     │  │   Stripe     │  │First Databank│          │
│  │   (SMS)      │  │ (Payments)   │  │ (Drug Data)  │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Auth0      │  │   SendGrid   │  │   Vimeo      │          │
│  │   (Auth)     │  │   (Email)    │  │  (Videos)    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Request Flow Example

**User Searches for Medication:**

```
1. User types "metformin" in search bar
   ↓
2. Mobile app sends GET /api/v1/medications/search?q=metformin
   ↓
3. API Gateway authenticates request (Auth0 JWT)
   ↓
4. Request routed to Medication Service
   ↓
5. Medication Service checks Redis cache
   ↓
6. Cache MISS → Query Elasticsearch for "metformin"
   ↓
7. Elasticsearch returns top 5 matches
   ↓
8. Medication Service fetches full details from PostgreSQL
   ↓
9. Response cached in Redis (TTL: 1 hour)
   ↓
10. JSON response sent to mobile app
   ↓
11. App renders medication summary
   ↓
12. Analytics event logged: "medication_searched"
```

---

## 🧩 System Components

### 1. Medication Service

**Responsibilities:**
- Serve medication information (summaries, side effects, interactions)
- Search and autocomplete functionality
- Content management (CRUD operations)
- Localization (English, Spanish)

**Endpoints:**
- `GET /api/v1/medications/search` - Search medications
- `GET /api/v1/medications/:id` - Get medication details
- `GET /api/v1/medications/:id/summary` - Plain-language summary
- `GET /api/v1/medications/:id/interactions` - Check interactions
- `GET /api/v1/medications/:id/side-effects` - Side effect details

**Technology Stack:**
- **Language:** Node.js (TypeScript)
- **Framework:** Express.js
- **Database:** PostgreSQL (relational data)
- **Cache:** Redis (frequently accessed data)
- **Search:** Elasticsearch (full-text search)

**Data Model:**
```typescript
interface Medication {
  id: string;
  genericName: string;
  brandNames: string[];
  drugClass: string;
  summary: {
    whatItDoes: string;
    howToTake: string;
    duration: string;
    sideEffects: {
      common: string[];
      serious: string[];
    };
  };
  interactions: {
    drugs: string[];
    foods: string[];
    conditions: string[];
  };
  content: {
    en: MedicationContent;
    es: MedicationContent;
  };
  lastUpdated: Date;
  reviewedBy: string; // Pharmacist ID
}
```

**Caching Strategy:**
- **L1 Cache (Redis):** Top 100 medications (90% of queries)
- **TTL:** 1 hour for medication details, 24 hours for static content
- **Cache Invalidation:** On content update (publish event)

---

### 2. User Service

**Responsibilities:**
- User authentication and authorization
- User profile management
- Medication list (user's current medications)
- Preferences and settings

**Endpoints:**
- `POST /api/v1/auth/register` - User registration
- `POST /api/v1/auth/login` - User login
- `GET /api/v1/users/:id` - Get user profile
- `PUT /api/v1/users/:id` - Update profile
- `GET /api/v1/users/:id/medications` - User's medication list
- `POST /api/v1/users/:id/medications` - Add medication to list

**Technology Stack:**
- **Language:** Node.js (TypeScript)
- **Framework:** Express.js
- **Database:** PostgreSQL
- **Auth:** Auth0 (delegated authentication)
- **Sessions:** Redis (session storage)

**Data Model:**
```typescript
interface User {
  id: string;
  email: string;
  passwordHash: string;
  profile: {
    firstName: string;
    lastName: string;
    dateOfBirth: Date;
    language: 'en' | 'es';
    timezone: string;
  };
  medications: UserMedication[];
  preferences: {
    reminderTime: string;
    notificationChannels: ('push' | 'sms' | 'email')[];
  };
  createdAt: Date;
  lastLogin: Date;
}

interface UserMedication {
  medicationId: string;
  prescribedDate: Date;
  duration: number; // days
  dosage: string;
  frequency: string;
  prescribedBy: string; // Doctor name
  pharmacy: string;
  refillsRemaining: number;
}
```

**Security:**
- Passwords hashed with bcrypt (cost factor: 12)
- JWT tokens for API authentication
- Refresh token rotation
- Rate limiting: 100 requests/hour per user

---

### 3. Pharmacist Q&A Service

**Responsibilities:**
- Receive and route patient questions
- AI-powered answer suggestions
- Pharmacist queue management
- Response delivery and notifications

**Endpoints:**
- `POST /api/v1/questions` - Submit question
- `GET /api/v1/questions/:id` - Get question details
- `GET /api/v1/questions/user/:userId` - User's question history
- `POST /api/v1/questions/:id/answer` - Pharmacist submits answer
- `GET /api/v1/pharmacists/queue` - Pharmacist dashboard (pending questions)

**Technology Stack:**
- **Language:** Python (for AI/ML components) + Node.js (API)
- **Framework:** Flask (Python), Express (Node)
- **Queue:** RabbitMQ (async processing)
- **AI:** OpenAI GPT-4 API (answer suggestions)
- **Database:** PostgreSQL

**Data Model:**
```typescript
interface Question {
  id: string;
  userId: string;
  medicationId: string;
  questionText: string;
  urgency: 'low' | 'medium' | 'high';
  aiSuggestedAnswer: string | null;
  pharmacistAnswer: string | null;
  answeredBy: string | null; // Pharmacist ID
  status: 'pending' | 'in_progress' | 'answered';
  createdAt: Date;
  answeredAt: Date | null;
  responseTime: number | null; // minutes
}
```

**AI Answer Suggestion Flow:**
1. User submits question
2. Question preprocessed (entity extraction, classification)
3. GPT-4 API call with context:
   - Medication details
   - User's medication history
   - Common Q&A database
4. AI suggests answer (shown immediately to user)
5. Question routed to pharmacist for review
6. Pharmacist approves/edits/rewrites answer
7. Final answer pushed to user (notification)

**SLA:**
- **AI Answer:** < 5 seconds
- **Pharmacist Review:** < 24 hours
- **Urgent Questions:** < 2 hours

---

### 4. Reminder Service

**Responsibilities:**
- Schedule and send medication reminders
- Multi-channel delivery (push, SMS, email)
- Reminder customization and snooze
- Treatment duration countdowns

**Endpoints:**
- `POST /api/v1/reminders` - Create reminder
- `PUT /api/v1/reminders/:id` - Update reminder
- `DELETE /api/v1/reminders/:id` - Delete reminder
- `POST /api/v1/reminders/:id/snooze` - Snooze reminder

**Technology Stack:**
- **Language:** Node.js
- **Queue:** RabbitMQ (delayed job queue)
- **Push:** Firebase Cloud Messaging (FCM)
- **SMS:** Twilio
- **Email:** SendGrid
- **Scheduler:** node-cron (recurring jobs)

**Data Model:**
```typescript
interface Reminder {
  id: string;
  userId: string;
  medicationId: string;
  schedule: {
    time: string; // "08:00"
    frequency: 'daily' | 'twice_daily' | 'weekly';
    daysOfWeek: number[]; // [0,1,2,3,4,5,6]
  };
  channels: ('push' | 'sms' | 'email')[];
  message: string;
  enabled: boolean;
  lastSent: Date | null;
  nextScheduled: Date;
}
```

**Reminder Flow:**
1. Scheduler checks for due reminders every minute
2. Queues reminder jobs in RabbitMQ
3. Worker processes job:
   - Fetch user preferences
   - Generate personalized message
   - Send via configured channels (push/SMS/email)
4. Log delivery status
5. Schedule next reminder

---

### 5. Analytics Service

**Responsibilities:**
- Event tracking and aggregation
- User behavior analysis
- Product metrics dashboards
- A/B test result tracking

**Endpoints:**
- `POST /api/v1/events` - Log event
- `GET /api/v1/analytics/dashboard` - Admin dashboard data
- `GET /api/v1/analytics/cohorts` - Cohort analysis
- `GET /api/v1/analytics/funnel` - Conversion funnel

**Technology Stack:**
- **Language:** Python (data processing)
- **Framework:** FastAPI
- **Event Stream:** Apache Kafka (real-time events)
- **Data Warehouse:** AWS Redshift (analytics)
- **Visualization:** Metabase (dashboards)

**Event Schema:**
```typescript
interface AnalyticsEvent {
  eventName: string;
  userId: string;
  sessionId: string;
  timestamp: Date;
  platform: 'ios' | 'android' | 'web';
  properties: {
    [key: string]: string | number | boolean;
  };
  context: {
    appVersion: string;
    deviceModel: string;
    osVersion: string;
    location: {
      country: string;
      city: string;
    };
  };
}
```

**Tracked Events:**
- `medication_searched`
- `medication_viewed`
- `video_watched`
- `quiz_completed`
- `question_submitted`
- `reminder_set`
- `treatment_completed`

**Metrics Pipeline:**
1. Client logs event → API Gateway
2. Event validated and enriched
3. Pushed to Kafka topic
4. Stream processors aggregate in real-time (Kafka Streams)
5. Batch jobs (hourly) write to Redshift
6. Dashboards query Redshift

---

### 6. Content Management Service

**Responsibilities:**
- Manage videos, images, and static assets
- Localization and translation workflows
- Content versioning and review
- CDN integration

**Endpoints:**
- `POST /api/v1/content/upload` - Upload media
- `GET /api/v1/content/:id` - Fetch content
- `PUT /api/v1/content/:id` - Update content
- `DELETE /api/v1/content/:id` - Delete content

**Technology Stack:**
- **Language:** Node.js
- **Storage:** AWS S3 (media files)
- **CDN:** AWS CloudFront (fast delivery)
- **Video:** Vimeo (hosting + transcoding)
- **Translation:** AWS Translate (initial) + human review

**Content Delivery:**
- **Images:** Served via CloudFront CDN (edge caching)
- **Videos:** Hosted on Vimeo, embedded in app
- **Static Assets:** Versioned URLs (cache busting)

---

## 🗄️ Database Design

### PostgreSQL Schema

#### Tables

**users**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  date_of_birth DATE,
  language VARCHAR(2) DEFAULT 'en',
  timezone VARCHAR(50) DEFAULT 'UTC',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  last_login TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
```

**medications**
```sql
CREATE TABLE medications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  generic_name VARCHAR(255) NOT NULL,
  brand_names TEXT[], -- Array of brand names
  drug_class VARCHAR(100),
  summary_text TEXT,
  how_to_take TEXT,
  side_effects_common TEXT[],
  side_effects_serious TEXT[],
  interactions_drugs TEXT[],
  interactions_foods TEXT[],
  content_en JSONB, -- Full English content
  content_es JSONB, -- Full Spanish content
  reviewed_by UUID REFERENCES pharmacists(id),
  last_updated TIMESTAMP DEFAULT NOW(),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_medications_generic_name ON medications(generic_name);
CREATE INDEX idx_medications_drug_class ON medications(drug_class);
CREATE INDEX idx_medications_updated ON medications(last_updated);
```

**user_medications**
```sql
CREATE TABLE user_medications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  medication_id UUID REFERENCES medications(id),
  prescribed_date DATE NOT NULL,
  duration_days INT,
  dosage VARCHAR(100),
  frequency VARCHAR(100),
  prescribed_by VARCHAR(255),
  pharmacy VARCHAR(255),
  refills_remaining INT DEFAULT 0,
  active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_meds_user_id ON user_medications(user_id);
CREATE INDEX idx_user_meds_active ON user_medications(active);
```

**questions**
```sql
CREATE TABLE questions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  medication_id UUID REFERENCES medications(id),
  question_text TEXT NOT NULL,
  urgency VARCHAR(10) CHECK (urgency IN ('low', 'medium', 'high')),
  ai_suggested_answer TEXT,
  pharmacist_answer TEXT,
  answered_by UUID REFERENCES pharmacists(id),
  status VARCHAR(20) CHECK (status IN ('pending', 'in_progress', 'answered')),
  created_at TIMESTAMP DEFAULT NOW(),
  answered_at TIMESTAMP,
  response_time_minutes INT
);

CREATE INDEX idx_questions_user_id ON questions(user_id);
CREATE INDEX idx_questions_status ON questions(status);
CREATE INDEX idx_questions_urgency ON questions(urgency);
CREATE INDEX idx_questions_created ON questions(created_at);
```

**reminders**
```sql
CREATE TABLE reminders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  medication_id UUID REFERENCES medications(id),
  reminder_time TIME NOT NULL,
  frequency VARCHAR(20),
  days_of_week INT[],
  channels VARCHAR(10)[], -- ['push', 'sms', 'email']
  message TEXT,
  enabled BOOLEAN DEFAULT TRUE,
  last_sent TIMESTAMP,
  next_scheduled TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_reminders_user_id ON reminders(user_id);
CREATE INDEX idx_reminders_next_scheduled ON reminders(next_scheduled);
CREATE INDEX idx_reminders_enabled ON reminders(enabled);
```

**pharmacists**
```sql
CREATE TABLE pharmacists (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  license_number VARCHAR(50) UNIQUE NOT NULL,
  license_state VARCHAR(2),
  specialties TEXT[],
  verified BOOLEAN DEFAULT FALSE,
  active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_pharmacists_license ON pharmacists(license_number);
CREATE INDEX idx_pharmacists_active ON pharmacists(active);
```

**analytics_events**
```sql
CREATE TABLE analytics_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_name VARCHAR(100) NOT NULL,
  user_id UUID,
  session_id UUID,
  timestamp TIMESTAMP DEFAULT NOW(),
  platform VARCHAR(20),
  properties JSONB,
  context JSONB
);

CREATE INDEX idx_events_name ON analytics_events(event_name);
CREATE INDEX idx_events_user ON analytics_events(user_id);
CREATE INDEX idx_events_timestamp ON analytics_events(timestamp);
```

### Database Partitioning Strategy

**analytics_events** (High Volume Table):
```sql
-- Partition by month for efficient querying and archival
CREATE TABLE analytics_events_2025_11 PARTITION OF analytics_events
  FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');

CREATE TABLE analytics_events_2025_12 PARTITION OF analytics_events
  FOR VALUES FROM ('2025-12-01') TO ('2026-01-01');
```

### Backup Strategy

- **Daily full backup** (automated, 3 AM UTC)
- **Continuous WAL archiving** (point-in-time recovery)
- **Retention:** 30 days rolling backups
- **Replication:** 1 read replica (for analytics queries)

---

## 🔌 API Design

### RESTful API Standards

**Base URL:** `https://api.medclear.com/v1`

**Authentication:**
```
Authorization: Bearer {JWT_TOKEN}
```

**Response Format:**
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "meta": {
    "requestId": "uuid",
    "timestamp": "2025-11-23T10:00:00Z"
  }
}
```

**Error Format:**
```json
{
  "success": false,
  "error": {
    "code": "MEDICATION_NOT_FOUND",
    "message": "Medication with ID abc123 not found",
    "details": {}
  },
  "meta": {
    "requestId": "uuid",
    "timestamp": "2025-11-23T10:00:00Z"
  }
}
```

### Key API Endpoints

#### Medication API

**Search Medications:**
```
GET /api/v1/medications/search?q=metformin&limit=10

Response:
{
  "success": true,
  "data": {
    "results": [
      {
        "id": "uuid",
        "genericName": "Metformin",
        "brandNames": ["Glucophage", "Fortamet"],
        "drugClass": "Biguanides"
      }
    ],
    "total": 15,
    "page": 1
  }
}
```

**Get Medication Details:**
```
GET /api/v1/medications/{id}

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "genericName": "Metformin",
    "brandNames": ["Glucophage"],
    "summary": {
      "whatItDoes": "Helps control blood sugar...",
      "howToTake": "Take with meals...",
      "duration": "As prescribed by doctor",
      "sideEffects": {
        "common": [
          "Nausea or upset stomach",
          "Diarrhea (especially when starting)"
        ],
        "serious": [
          "Severe stomach pain",
          "Unusual muscle pain"
        ]
      }
    }
  }
}
```

**Check Drug Interactions:**
```
POST /api/v1/medications/{id}/check-interactions

Body:
{
  "medications": ["ibuprofen", "vitamin-d"],
  "foods": ["alcohol"]
}

Response:
{
  "success": true,
  "data": {
    "interactions": [
      {
        "substance": "alcohol",
        "severity": "moderate",
        "description": "May increase risk of lactic acidosis",
        "recommendation": "Limit alcohol consumption"
      }
    ]
  }
}
```

#### User API

**Register User:**
```
POST /api/v1/auth/register

Body:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "dateOfBirth": "1985-05-15"
}

Response:
{
  "success": true,
  "data": {
    "userId": "uuid",
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token"
  }
}
```

**Add Medication to User List:**
```
POST /api/v1/users/{userId}/medications

Body:
{
  "medicationId": "uuid",
  "prescribedDate": "2025-11-20",
  "duration": 30,
  "dosage": "500mg",
  "frequency": "Twice daily"
}

Response:
{
  "success": true,
  "data": {
    "userMedicationId": "uuid",
    "medication":
