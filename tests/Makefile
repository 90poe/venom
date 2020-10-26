


TEST_REDIS_PORT =  $(if ${VENOM_TEST_REDIS_PORT},${VENOM_TEST_REDIS_PORT},6379)
TEST_REDIS_START = docker run -d -p $(TEST_REDIS_PORT):6379 --name redis-venom registry.ovh.net/official/redis:3.2.5

TEST_PG_PORT = $(if ${VENOM_TEST_PG_PORT},${VENOM_TEST_PG_PORT},5432)
TEST_PG_START = docker run -d -p $(TEST_PG_PORT):5432 -e POSTGRES_PASSWORD=venom -e POSTGRES_USER=venom -e POSTGRES_DB=venom --name pg-venom postgres

TEST_MYSQL_PORT =  $(if ${VENOM_TEST_MYSQL_PORT},${VENOM_TEST_MYSQL_PORT},3306)
TEST_MYSQL_START = docker run -d -p $(TEST_MYSQL_PORT):3306 -e MYSQL_PASSWORD=venom -e MYSQL_USER=venom -e MYSQL_DATABASE=venom --name mysql-venom mysql/mysql-server


TEST_REDIS_HOST =  $(if ${VENOM_TEST_REDIS_HOST},${VENOM_TEST_REDIS_HOST},localhost)
TEST_MYSQL_HOST =  $(if ${VENOM_TEST_MYSQL_HOST},${VENOM_TEST_MYSQL_HOST},localhost)
TEST_PG_HOST = $(if ${VENOM_TEST_PG_HOST},${VENOM_TEST_PG_HOST},localhost)



test-docker-start:
	$(TEST_REDIS_START)
	$(TEST_PG_START)
	$(TEST_MYSQL_START)
	@sleep 15

clean:
	docker stop redis-venom; docker rm redis-venom; exit 0
	docker stop mysql-venom; docker rm mysql-venom; exit 0
	docker stop pg-venom; docker rm pg-venom; exit 0
	docker stop smtp-venom; docker rm smtp-venom; exit 0


test: clean test-docker-start
	venom run . \
		--var redisHost=localhost \
		--var smtpPort=${TEST__SMTP_PORT} \
		--var redisPort=${TEST_REDIS_PORT} \
		--var mysqlHost=localhost \
		--var mysqlPort=${TEST_MYSQL_PORT} \
		--var pgHost=localhost \
		--var pgPort=${TEST_PG_PORT} \
		--exclude MyTestSuiteWeb.yml \
		--exclude MyTestSuiteSMTP.yml \
		--exclude MyTestSuiteImap.yml \
		--exclude MyTestSuiteSSH.yml \
		* \
		--details high

TMP_DIR = /tmp/ovh/venom
VENOM_TMP_BIN = $(TMP_DIR)/venom
DOCK_COMP_CMD = docker-compose -f kafka/docker-compose.yml

# debug with
# kt consume -topic test-topic
# kt consume -topic test-topic-avro
.PHONY: local-env-start
local-env-start:
	$(DOCK_COMP_CMD) up \
		--force-recreate \
		--remove-orphans \
		zookeeper kafka0 schema-registry || (docker ps -a -q | xargs docker rm -vf ; exit 1)

.PHONY: local-env-stop
local-env-stop:
	$(DOCK_COMP_CMD) down --volumes --remove-orphans
	docker image prune --force --filter label=stage=test
	docker images -aq -f 'dangling=true' | xargs docker rmi

# use
# vars:
#   kafkaHost: 'localhost'
#   kafkaPort: '9092'
.PHONY: test-kafka
test-kafka: $(VENOM_TMP_BIN) test-kafka-results
	$(VENOM_TMP_BIN) run \
		--log debug \
		--parallel 5 \
		--format xml \
		--output-dir "results" \
		--strict \
		MyTestSuiteKafka.yml \
		MyTestSuiteKafkaAvro.yml

$(VENOM_TMP_BIN):
	go mod download
	go mod vendor
	CGO_ENABLED=0 go build \
		-mod=vendor \
		-o $(VENOM_TMP_BIN) \
		../cli/venom/.
	chmod +x $(VENOM_TMP_BIN)

test-kafka-results:
	mkdir -p results
