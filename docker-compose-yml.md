version: '3.4'

services:
  exercise.wwwapi:
    image: ${DOCKER_REGISTRY-}exercisewwwapi
    container_name: students.api
    build:
      context: .
      dockerfile: exercise.wwwapi/Dockerfile
    ports:
      - "5000:5000"
      - "5001:5001"

  students.database:
    image: postgres:latest
    container_name: workshop.database
    environment:
        - POSTGRES_DB=postgres
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=password
    volumes:
        - ./.containers-db:/var/lib/postgresql/data
    ports:
        - 5432:5432
  students.cache:
    image: redis:latest
    restart: always
    ports:
        - '6379:6379'




version: '3.4'

services:
  exercise.wwwapi:
    image: ${DOCKER_REGISTRY-}exercisewwwapi
    container_name: students.api
    build:
      context: .
      dockerfile: exercise.wwwapi/Dockerfile
    ports:
      - "5000:5000"
      - "5001:5001"
    networks:
      - sibbs-network
    depends_on:
      workshop.database:
        condition: service_healthy

  workshop.database:
    image: postgres:latest
    container_name: workshop.database
    environment:
        - POSTGRES_DB=postgres
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=password
    volumes:
        - ./.containers-db:/var/lib/postgresql/data
    ports:
        - 5432:5432
    networks:
        - sibbs-network
    healthcheck:
        test: ["CMD", "pg_isready", "-q", "-d", "postgres", "-U", "postgres"]
        interval: 1s
        timeout: 5s
        retries: 5

  
  students.cache:
    image: redis:latest
    restart: always
    ports:
        - '6379:6379'
volumes:
  postgres_data:
    driver: local
networks:
  sibbs-network:
    driver: bridge