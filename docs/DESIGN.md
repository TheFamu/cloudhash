# Design Doc

## Objective

The goal is to write a complex data structure store with client and server in a
single go binary that's easily deployable to aws or to self-host locally.

## Stack

* Go Lang: Server & Client
* Cloudflare Turnstile
* AWS: Lambda, DynamoDB, SSM Parameter Store
* Self Hosted: SQLite
* Terraform
* Github Actions

## Features

* Can run the server either self-hosted or deploy to aws via terraform.
* Client & server in one binary.

## Endpoints

* Three Endpoint
  - `/`     (html)
  - `/api`  (rest)
  - `/keys` (post)
* Home Page
  - `GET /`: HTML page w/ Cloudflare Turnstile (captcha disabled for self-hosted)
* Request API Key
  - `POST /keys`: Request a new API key (protected)
* API Actions
  - `GET /api`: List all datasets 
  - `GET /api/{name}`: Get dataset
  - `PUT /api/{name}`: Create or Update dataset
  - `DELETE /api/{name}`: Delete dataset

## CLI Interface

URL defaults to cloud instance, overwritten by `CHASH_URL` env var or `--url <url>` flag.

API key passed via `CHASH_KEY` env var.

CLI utils parameters map to API actions.

* `chash list` -> `GET /api`
* `chash set DATASET_NAME < config.json` -> `PUT /api/{name}`
* `chash get DATASET_NAME` -> `GET /api/{name}`
* `chash delete DATASET_NAME` -> `DELETE /api/{name}`

The CLI util should accept data via stdin or `-d <data>` / `--data <data>` flags for `set`.

### Extra Flags

* `-o <file>` / `--output <file>`: Write output to a file.

## Cloud Abuse Considerations & Remediations

We really don't want to give Bezos any money. So we're taking several
steps to ensure access to the public API is restricted / limited:

1. App endpoints only accessible via cloudflare proxy w/ secret header.
  - Also gives us rate-limiting and bot protection for free.
2. Every API route protected via an API key.
  - Key hashes stored in DynamoDB (or SQLite for self-hosted).
  - Key only shown once.
  - Key daily quota kept in DynamoDB (unlimited for self-hosted).
3. Cloud limited to 128 MB per request body (unlimited for self-hosted).
  - Limit subject to change.
4. Cloud limited to 512 MB total storage per key (unlimited for self-hosted).
  - Limit subject to change.
5. Cloud limited to 1000 requests per day (unlimited for self-hosted).
  - Limit subject to change.
6. Cloud limit Lambda Concurrency to low value (10) to prevent abuse.
7. AWS Budget alerts at $1.
  - Possibly even wire through SNS to a small function that sets concurrency to 0 (as kill switch).

### Request Key Flow

How users get keys for public API.

```
       Home Page Get Key Form
                |
                v
    Cloudflare Turnstile (Captcha)
                |
                v
    App Create Key -> Hash -> DB
                |
                v
    Returns Key + Usage Limits Info
```

## One Binary, Three Modalities

The go binary can do three things:

1. Run as server in AWS Lambda.
  - Can use `AWS_LAMBDA_FUNCTION_NAME` env var to tell if we're in lambda mode.
2. Run as server in self-hosted mode.
  - Default if `AWS_LAMBDA_FUNCTION_NAME` not present and run with `serve` argument.
  - Will use local SQLite DB instead of DynamoDB.
3. Run as client to connect to server.

We can use a switch statement inside of main to determine what mode to run in
and what internal flags to set.

## GitHub Actions

Github actions should do a few things:

* Run linter against codebase
* Run tests against amd64 binary (and arm binary somehow???)
* Compile `linux/arm64` build named `bootstrap` for AWS Lambda.
* Zip `bootstrap` for upload to Lambda.
* Compile `linux/amd64` build named `chash` for Github Releases.
* Run terraform to apply changes to aws.

## Questions

* Should we use OAuth 2.0 "Signin with GitHub" to access api key page?
  - Would cut down on Bot spam, might not be necessary.

## Resources

https://developers.cloudflare.com/turnstile/

https://aws.amazon.com/dynamodb/

https://docs.aws.amazon.com/lambda/latest/dg/lambda-functions-chapter.html

