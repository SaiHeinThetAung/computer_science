services:
  janus:
    build:
      context: .
      dockerfile: Dockerfile
    network_mode: "host"
    environment:
      - PUBLIC_IP=${PUBLIC_IP}
    volumes:
      - recordings:/opt/janus/recordings
    restart: unless-stopped

volumes:
  recordings: