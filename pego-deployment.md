# Suitcase on Pego Network
This project is a fork from Gnosis Safe Global. The Suitcase project consists of the following 8 repositories. To deploy the project, go to each repo by following the sequence below and refer to steps stated in `pego-deployment.md`, which can be found in each repo.

1. [safe-singleton-factory](https://github.com/Pego-DAO/safe-singleton-factory)
2. [suitcase-contract](https://github.com/Pego-DAO/suitcase-contract)
3. **[safe-transaction-service](https://github.com/Pego-DAO/safe-transaction-service)** - Currently viewing
4. [safe-client-gateway](https://github.com/Pego-DAO/safe-client-gateway)
5. [safe-config-service](https://github.com/Pego-DAO/safe-config-service)
6. [suitcase-deployment](https://github.com/Pego-DAO/suitcase-deployment)
7. [suitcase-sdk](https://github.com/Pego-DAO/suitcase-sdk)
8. [suitcase-ui](https://github.com/Pego-DAO/suitcase-ui)

## safe-transaction-service
*Prerequisite: Smart contract in [suitcase-contract](https://github.com/Pego-DAO/suitcase-contract) have deployed successfully to the target network*

1. Create a `.env` file and copy the following code and provide value accordingly.
```
PYTHONPATH=/app/
DJANGO_SETTINGS_MODULE=config.settings.production
DEBUG=0
ROOT_LOG_LEVEL=INFO
NGINX_HOST_PORT=80
DATABASE_URL=psql://postgres:postgres@db:5432/postgres
ETHEREUM_NODE_URL=https://pegorpc.com # Node rpc
ETHEREUM_TRACING_NODE_URL=
ETH_L2_NETWORK=1
ETH_INTERNAL_NO_FILTER=1
REDIS_URL=redis://redis:6379/0
CELERY_BROKER_URL=amqp://guest:guest@rabbitmq/
DJANGO_SECRET_KEY=lrlIvEVRxHimueOmPk2jUExxxxxx # Generate a random string and will be used by other services
DJANGO_ALLOWED_HOSTS=*
CSRF_TRUSTED_ORIGINS=https://transaction.suitcase.plus,https://config.suitcase.plus,https://gateway.suitcase.plus,http://127.0.0.1:3333,http://127.0.0.1:3000 # url of safe transaction, gateway service, config service, http://127.0.0.1:3333 and http://127.0.0.1:3000

```

2. Update source of pego price
```
# In file safe_transaction_service/tokens/clients/coingecko_client.py,
# search for the code snippet bleow and update Coingecko's API ID.
def get_peg_usd_price(self) -> float:
    return self.get_price("pego-network-2")

# In file safe_transaction_service/tokens/services/price_service.py,
# search for the code snippet below and update chain id accordingly.
def get_native_coin_usd_price(self) -> float:
    ...
    elif self.ethereum_network in (
        123456,
        20201022
    ):
        return self.get_peg_usd_price()
    ...
```

3. Run this repo with docker
```
# To build docker
docker-compose build --force-rm

# Bring up docker container
docker-compose up -d

# To view docker process
docker ps -a
```
*Note: Some proccess might exited due to database dependencies. Check the docker processes and run `docker-compose up -d` to restart exited processes after database has created.*

4. Create an account to login transaction service,
```
docker exec -it <web-docker-container-id> python manage.py createsuperuser
```

5. Login to transaction service admin site at `<site_url>/admin`, and provide value to the following menu,
    - `Chains` at `<site_url>/admin/history/chain/`, add chain Id.
    - `Proxy factories` at `<site_url>/admin/history/proxyfactory/`, add proxy factory contract address and contract creation block number.
    - `Safe master copies` at `<site_url>/admin/history/safemastercopy/`, GnosisSafe contract address, contract creation block number and version as 1.3.0.
    - `Safe master copies` at `<site_url>/admin/history/safemastercopy/`, GnosisSafeL2 contract address, contract creation block number, version as 1.3.0 and select L2.