# Todos

* [ ] **Write API Key Generation Code**
  - Needs to hash key and store to DB via DataStore interface

* [ ] **Write API Endpoint Code**
  - See endpoints list in [DESIGN.md](docs/DESIGN.md)
  - Needs to use DataStore interface to talk to storage backends

* [ ] **Write DataStore (neutral) Interface Code**
  - Interface that can talk to DynamoDB or SQLite

* [ ] **Write Lambda Server(less) Code**
  - Needs to talk to DynamoDB via DataStore inteface
  - Needs to enforce limits on cloud usage

* [ ] **Write Self-Hosted Server Code**
  - Needs to talk to SQLite DB via DataStore interface

* [ ] **Write CLI Client Code**
  - Needs to handle argv
  - Needs to handle opt args
  - Needs to handle opt stdin
  - Needs to make API calls

* [ ] **Write GitHub Actions CI/CD Code**
  - Needs to lint, compile bins, run test, terraform apply

* [ ] **Write Go Lang Test Code**
  - Needs to run unit tests

* [ ] **Write Terraform Apply Code**
  - Needs to provision infra, upload bin, set secrets
