# Twitter Personality Analysis Real-Time Pipeline
A containerized real-time data pipeline that classifies Twitter personalities using MBTI analysis. Features streaming architecture with Python, MQTT, Kafka, and MySQL, delivering live insights through interactive Superset dashboards.

## Architecture Overview

The pipeline uses a MySQL-based architecture for optimal compatibility and reliability:

```
Twitter Data → Python Publisher → MQTT Broker → Kafka Bridge → MySQL Database → Superset Dashboard
```

### Core Services

| Service | Purpose | Port |
|---------|---------|------|
| Python Publisher | Data ingestion & MQTT publishing | - |
| MQTT Broker | Message broker for real-time streaming | 1883 |
| MQTT-Kafka Bridge | Bridge MQTT messages to Kafka | - |
| Zookeeper | Kafka coordination service | 2181 |
| Kafka Broker | Distributed streaming platform | 9092 |
| MySQL Database | Primary data storage | 3306 |
| MySQL Consumer | Kafka to MySQL data pipeline | - |
| Redis Cache | Superset caching layer | 6379 |
| Superset | Business intelligence dashboard | 8088 |

## Data Sources

- **tweets1.json**: 8,328 users with tweet collections
- **users1.json**: User profile information and metadata  
- **edges1.json**: User connection and relationship data
- **mbti_labels.csv**: MBTI personality type classifications (16 types)

## Quick Start

### Prerequisites

- Docker Desktop 4.0+ with Docker Compose
- 8GB+ RAM (recommended 12GB)
- 10GB+ free disk space
- Available ports: 1883, 3306, 6379, 8088, 9092, 2181

### Deployment

1. Navigate to project directory:
   ```powershell
   cd twitter_app
   ```

2. Deploy complete pipeline:
   ```powershell
   docker-compose -f docker-compose-mysql.yml up -d
   ```

3. Wait 2-3 minutes for services to initialize, then verify:
   ```powershell
   # Check all services are running
   docker ps

   # Verify data is flowing
   docker exec mysql mysql -u twitter_user -ptwitter_password twitter_analytics -e "SELECT COUNT(*) FROM tweets;"

   # Check real-time processing
   docker logs python-publisher --tail 5
   docker logs mqtt-kafka-bridge --tail 5
   docker logs kafka-mysql-consumer --tail 5
   ```


4. **Alternative: Manual step-by-step deployment**
   ```powershell
   # Step 1: Start core infrastructure
   docker-compose -f docker-compose-mysql.yml up -d zookeeper kafka mysql redis

   # Step 2: Start messaging services
   docker-compose -f docker-compose-mysql.yml up -d mosquitto python-publisher

   # Step 3: Start data pipeline
   docker-compose -f docker-compose-mysql.yml up -d mqtt-kafka-bridge kafka-mysql-consumer

   # Step 4: Start dashboard
   docker-compose -f docker-compose-mysql.yml up -d superset
   ```

### Service Access

| Service | URL | Credentials |
|---------|-----|-------------|
| Superset Dashboard | http://localhost:8088 | admin/admin |
| MySQL Database | localhost:3306 | twitter_user/twitter_password |

## Data Pipeline Flow

### Real-Time Processing

1. **Data Ingestion**: Python Publisher processes Twitter JSON files, enriches with MBTI labels, publishes to MQTT every 2 seconds
2. **Message Streaming**: MQTT-Kafka Bridge forwards messages with user_id partitioning
3. **Data Persistence**: Kafka Consumer validates and stores data in MySQL with analytics views
4. **Visualization**: Superset creates real-time dashboards with 30-second refresh intervals

### Message Format

```json
{
  "user_id": 1234567890,
  "screen_name": "example_user",
  "tweet": "Sample tweet content...",
  "timestamp": "2025-07-13 18:32:33",
  "location": "New York, NY",
  "verified": false,
  "statuses_count": 1250,
  "mbti_personality": "infj",
  "total_retweet_count": 45,
  "total_favorite_count": 128
}
```

## Database Schema

### Primary Table Structure

```sql
CREATE TABLE tweets (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    screen_name VARCHAR(255),
    tweet TEXT,
    timestamp DATETIME,
    location VARCHAR(255),
    verified BOOLEAN DEFAULT FALSE,
    statuses_count BIGINT DEFAULT 0,
    mbti_personality VARCHAR(10),
    total_retweet_count BIGINT DEFAULT 0,
    total_favorite_count BIGINT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_mbti (mbti_personality),
    INDEX idx_timestamp (timestamp)
);
```

## Dashboard Configuration

### Superset Setup

1. Access http://localhost:8088 (admin/admin)
2. Add database connection: `mysql://twitter_user:twitter_password@mysql:3306/twitter_analytics`
3. Create charts using queries from `DASHBOARD_QUERIES.md`

### Key Visualizations

- **MBTI Distribution**: Bar chart showing personality type popularity
- **Recent Feed**: Table showing live tweet stream with metadata
- **Personality Breakdown**: Pie chart analyzing extrovert/introvert patterns


## Configuration Files

```
project/
├── docker-compose-mysql.yml              # Main deployment configuration
├── config/
│   ├── mosquitto/mosquitto.conf          # MQTT broker settings
│   ├── superset-mysql/superset_config.py # Dashboard configuration
│   └── mysql/init.sql                    # Database schema
├── python-publisher/publisher.py         # Data ingestion service
├── mqtt-kafka-bridge/bridge.py           # Message routing service
├── kafka-mysql-consumer/consumer.py      # Data persistence service
└── dashboard-queries.sql                 # Pre-built analytics queries
```

## Technical Implementation

### MQTT-Kafka Bridge

Custom Python bridge implementation provides:
- Automatic reconnection on failures
- Message deduplication using user_id as key
- Real-time error logging and monitoring
- Configurable retry logic and backoff

### Performance Characteristics

- **Processing Rate**: 30 tweets/minute (1 every 2 seconds)
- **End-to-end Latency**: < 5 seconds
- **Memory Usage**: ~6GB total across all services
- **Data Volume**: 1,175+ tweets processed continuously
- **Coverage**: All 16 MBTI personality types represented

## Deployment Management

### Start/Stop Commands

```powershell
# Start pipeline
docker-compose -f docker-compose-mysql.yml up -d

# Monitor processing
docker logs python-publisher --follow

# Stop pipeline
docker-compose -f docker-compose-mysql.yml down

# Stop and remove all data
docker-compose -f docker-compose-mysql.yml down -v
```

## Proof of Concept Results

### Successful Deployment
![Docker Services Running](proof_screen_shots/docker_desktop_containers.PNG)
*All 9 Docker services running in production configuration*

### Functional Dashboard
![Superset Dashboard](proof_screen_shots/superset_main.PNG)
*Superset dashboard with full functionality and database connectivity*

![Dashboard Visualizations](proof_screen_shots/dashboard_twitter.PNG)
*Professional charts displaying real-time MBTI personality analysis*

