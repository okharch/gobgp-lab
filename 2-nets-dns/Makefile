.PHONY: all up down clean \
        ping2 tracet2 \
        ping1 trace1

all: up

up:
	@echo "Bringing up the network with Docker Compose..."
	docker compose up -d

down:
	@echo "Tearing down the network..."
	docker compose down

clean: down

ping2:
	@echo "Sending ping from host1 (172.20.0.100) to host2 (172.20.1.100) by name..."
	docker exec -it host1 ash -c "echo 'Pinging host2 (172.20.1.100) from host1...'; ping -c 3 host2"

trace2:
	@echo "Performing traceroute from host1 (172.20.0.100) to host2 (172.20.1.100) by name..."
	docker exec -it host1 ash -c "traceroute host2"

ping1:
	@echo "Sending ping from host2 (172.20.1.100) to host1 (172.20.0.100) by name..."
	docker exec -it host2 ash -c "echo 'Pinging host1 (172.20.0.100) from host2...'; ping -c 3 host1"

trace1:
	@echo "Performing traceroute from host2 (172.20.1.100) to host1 (172.20.0.100) by name..."
	docker exec -it host2 ash -c "traceroute host1"