create an interactive chatbot application that generates YAML files based on client expectations, integrating TTS (Text-to-Speech), STT (Speech-to-Text), and other APIs, follow these steps:

### 1. **Frontend Setup**

- **HTML & CSS**: Design a simple interface with a chatbox for text input/output and buttons for audio interaction.
- **JavaScript**: Use Web APIs to handle audio recording and playback.

### 2. **Backend Setup**

- Use Node.js and Express for a lightweight server.
- Integrate a database (e.g., MongoDB) to save client templates and configurations.

### 3. **AI and API Integration**

- **Speech Recognition (STT)**: Use Web Speech API for real-time conversion of speech to text.
- **Text-to-Speech (TTS)**: Use Web Speech Synthesis API for converting text responses from the bot to speech.
- **Language Model API**: Use an AI model (such as OpenAI's API) to generate responses and YAML structures based on text inputs.

### 4. **Pipeline Generation Logic**

- Create a customizable template for YAML pipelines.
- Use JavaScript functions to populate the templates with user input received from the chatbot.

### 5. **Chatbot Flow**

- **Greeting**: Introduce the bot’s capabilities.
- **Inquiry**: Ask the client about their needs (e.g., monitoring system type, input sources).
- **Configuration**: Collect details iteratively (e.g., RTSP URLs, required processes).
- **YAML Generation**: Build a YAML file using the provided template by integrating client inputs into it.
- **Feedback & Download**: Allow clients to review and download the generated YAML.

### 6. **Example Chat Flow**

1. **Bot**: "Hello! What type of monitoring system would you like to configure?"
2. **Client**: "I need a security monitoring setup."
3. **Bot**: "Great! What are the camera input URLs?"
4. **Client**: "Camera 1: rtsp://camera1.local/entrance and Camera 2: rtsp://camera2.local/parking."
5. **Bot**: "What objects are you interested in detecting?"
6. **Client**: "People and vehicles."

### 7. **Sample YAML File**
Przykłady pokazują różne scenariusze użycia:

    Security Monitoring:

    Monitoring kamer
    Detekcja obiektów
    Kontrola dostępu

    Production Monitoring:

    Kontrola jakości
    Monitoring maszyn
    Dane z czujników

    Smart Home:

    Automatyzacja
    Kamery
    Czujniki IoT

    Social Media Analytics:

    Analiza sentymentu
    Wykrywanie trendów
    Agregacja danych

    Infrastructure Monitoring:

    Analiza logów
    Metryki sieciowe
    Alerty

Każdy pipeline pokazuje:

    Różne źródła danych
    Różne typy przetwarzania
    Różne formaty wyjściowe
    Różne systemy powiadomień

Można łatwo dostosować te przykłady przez:

    Zmianę parametrów
    Dodanie nowych tasków
    Modyfikację callbacków
    Dodanie nowych protokołów


Here's a basic example of how the YAML might look:

```yaml
# config/pipelines.yaml

pipelines:
  # System monitoringu bezpieczeństwa
  - name: security_monitoring
    startup:
      - grpc://detector:50051/start?model=yolov5&confidence=0.6
      - grpc://face-detector:50052/start?model=face_recognition&min_size=80
    tasks:
      - input: rtsp://camera1.local:554/entrance?fps=15
        process: grpc://detector:50051/detect_objects?classes=person,vehicle
        callback: grpc://alert-service:50052/RegisterCallback

      - input: rtsp://camera2.local:554/parking?fps=10
        process: grpc://detector:50051/detect_objects?classes=car,truck,bicycle
        callback: grpc://alert-service:50052/RegisterCallback

      - input: rtsp://camera3.local:554/reception?fps=5
        process: grpc://face-detector:50052/recognize_faces
        callback: grpc://access-control:50053/RegisterCallback

    callback:
      "grpc://alert-service:50052/RegisterCallback":
        "grpc://localhost:50051/convertData":
          - file:///var/log/security/detections.json?mode=append
        "grpc://localhost:50051/convertDataForAlerts":
          - rss://security.local:8080/alerts?format=json&max_items=1000
          - webhook://slack.com/api/security-alerts?token=${SLACK_TOKEN}

  # System monitorowania produkcji
  - name: production_monitoring
    startup:
      - grpc://quality-detector:50055/start?model=defect_detection&threshold=0.8
      - grpc://metrics-collector:50056/start?interval=1s
    tasks:
      - input: rtsp://line1-camera/feed?fps=30
        process: grpc://quality-detector:50055/detect_defects
        callback: grpc://production-control:50057/QualityCallback

      - input: mqtt://sensors/temperature/+?interval=1s
        process: grpc://metrics-collector:50056/analyze_metrics
        callback: grpc://monitoring:50058/MetricsCallback

      - input: modbus://plc1/status?interval=100ms
        process: grpc://metrics-collector:50056/process_plc_data
        callback: grpc://monitoring:50058/PLCCallback

    callback:
      "grpc://production-control:50057/QualityCallback":
        "grpc://converter:50051/convertToMQTT":
          - mqtt://production/quality/line1?retain=true&qos=1
        "grpc://converter:50051/convertToDatabase":
          - postgresql://timescale:5432/metrics?table=quality_metrics

      "grpc://monitoring:50058/MetricsCallback":
        "grpc://converter:50051/convertToPrometheus":
          - prometheus://pushgateway:9091/metrics/job/production
        "grpc://converter:50051/convertToInflux":
          - influxdb://influx:8086/write?db=production&precision=ms

  # System IoT i smart home
  - name: smart_home_automation
    startup:
      - grpc://automation-engine:50060/start?config=home_rules
      - grpc://presence-detector:50061/start?sensitivity=high
    tasks:
      - input: mqtt://zigbee2mqtt/+/+/state?retain=true
        process: grpc://automation-engine:50060/process_state
        callback: grpc://home-control:50062/StateCallback

      - input: rtsp://doorbell-camera/stream?fps=10
        process: grpc://presence-detector:50061/detect_presence
        callback: grpc://notification:50063/PresenceCallback

      - input: mqtt://weather/outdoor/+?interval=5m
        process: grpc://automation-engine:50060/process_weather
        callback: grpc://home-control:50062/WeatherCallback

    callback:
      "grpc://home-control:50062/StateCallback":
        "grpc://converter:50051/convertToHomeAssistant":
          - mqtt://homeassistant/state/+?retain=true
        "grpc://converter:50051/convertToHistory":
          - influxdb://influx:8086/write?db=home_history

      "grpc://notification:50063/PresenceCallback":
        "grpc://converter:50051/convertToNotification":
          - pushover://user/notify?priority=high
          - telegram://bot/send?chat_id=${CHAT_ID}
        "grpc://converter:50051/convertToStorage":
          - s3://bucket/presence-events/

  # System analizy mediów społecznościowych
  - name: social_media_analytics
    startup:
      - grpc://sentiment-analyzer:50070/start?model=bert&language=multi
      - grpc://trend-detector:50071/start?interval=1m
    tasks:
      - input: twitter://api/stream?keywords=brand,product
        process: grpc://sentiment-analyzer:50070/analyze
        callback: grpc://analytics:50072/SentimentCallback

      - input: rss://news.feed/tech?interval=15m
        process: grpc://sentiment-analyzer:50070/analyze
        callback: grpc://analytics:50072/NewsCallback

      - input: websocket://reddit/stream?subreddits=technology,programming
        process: grpc://trend-detector:50071/detect_trends
        callback: grpc://analytics:50072/TrendCallback

    callback:
      "grpc://analytics:50072/SentimentCallback":
        "grpc://converter:50051/convertToElastic":
          - elasticsearch://elastic:9200/sentiment
        "grpc://converter:50051/convertToSlack":
          - webhook://slack/marketing?token=${SLACK_TOKEN}

      "grpc://analytics:50072/TrendCallback":
        "grpc://converter:50051/convertToVisualization":
          - grafana://dashboard/trends?key=${GRAFANA_KEY}
        "grpc://converter:50051/convertToEmail":
          - smtp://mail/send?to=team@company.com

  # System monitorowania infrastruktury
  - name: infrastructure_monitoring
    startup:
      - grpc://log-analyzer:50080/start?patterns=error,warning,critical
      - grpc://metric-collector:50081/start?interval=30s
    tasks:
      - input: syslog://servers/+/system?facility=*
        process: grpc://log-analyzer:50080/analyze_logs
        callback: grpc://monitoring:50082/LogCallback

      - input: snmp://network/+/metrics?community=public
        process: grpc://metric-collector:50081/collect_metrics
        callback: grpc://monitoring:50082/NetworkCallback

      - input: prometheus://scrape/targets/*?interval=15s
        process: grpc://metric-collector:50081/process_metrics
        callback: grpc://monitoring:50082/MetricsCallback

    callback:
      "grpc://monitoring:50082/LogCallback":
        "grpc://converter:50051/convertToELK":
          - elasticsearch://elastic:9200/logs
          - kibana://dashboard/system-logs
        "grpc://converter:50051/convertToPagerDuty":
          - pagerduty://api/event?severity=high

      "grpc://monitoring:50082/NetworkCallback":
        "grpc://converter:50051/convertToTimescale":
          - postgresql://timescale:5432/network_metrics
        "grpc://converter:50051/convertToGrafana":
          - grafana://dashboard/network?uid=network-overview
```

### Tools and Technologies

- **Frontend**: HTML, CSS, JavaScript, Web Audio API.
- **Backend**: Node.js, Express, MongoDB.
- **APIs**: Web Speech API for TTS and STT, AI language models.

This setup allows seamless interactions, enabling clients to configure complex systems easily through an intuitive chat interface with window for yaml file generated during conversation.
Prepare docker compose and .env file with init.sh script to prepare environment, and start.sh test.sh to test everything, start.sh to start app
build necessary tests and curl commands in test.sh to check API 
