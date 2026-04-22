# AWS SESv2 Mock

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Docker AMD64](https://img.shields.io/badge/docker-amd64-blue)
![Docker ARM64](https://img.shields.io/badge/docker-arm64-green)
![Build](https://img.shields.io/github/actions/workflow/status/hatchddigital/aws-sesv2-mock/docker.yml?branch=master)

A local mock server for the **AWS SESv2** `SendEmail` API, useful for running integration tests and local development without hitting real AWS infrastructure. Incoming messages are written to disk so you can inspect the raw payload, headers and body.

This is a [Hatchd Digital](https://hatchd.com.au) fork of [askrella/aws-ses-mock](https://github.com/askrella/aws-ses-mock), upgraded to support the SESv2 JSON API (the v1 project targets the legacy SES query API).

# :gear: Getting Started

## Running the Docker Container

```bash
docker run -p 8081:8081 ghcr.io/hatchddigital/aws-sesv2-mock:latest
```

The server listens on port `8081` by default. Override with the `PORT` environment variable. Emails are written to the working directory unless `OUTPUT_DIR` is set.

```bash
docker run -p 8081:8081 \
  -e OUTPUT_DIR=/mail \
  -v $(pwd)/mail:/mail \
  ghcr.io/hatchddigital/aws-sesv2-mock:latest
```

## Usage with NodeJS

Using the AWS SDK v3 SESv2 client, point the `endpoint` at the mock:

```javascript
import { SESv2Client, SendEmailCommand } from '@aws-sdk/client-sesv2'

const ses = new SESv2Client({
  region: 'us-east-1',
  endpoint: 'http://localhost:8081',
})

await ses.send(new SendEmailCommand({
  FromEmailAddress: 'sender@example.com',
  Destination: { ToAddresses: ['recipient@example.com'] },
  Content: {
    Simple: {
      Subject: { Data: 'Test email' },
      Body: { Text: { Data: 'Hello from the mock!' } },
    },
  },
}))
```

## Usage with Golang

Using the AWS SDK v2 SESv2 client, override the endpoint resolver:

```golang
customResolver := aws.EndpointResolverWithOptionsFunc(func(service, region string, options ...interface{}) (aws.Endpoint, error) {
    if overrideEndpoint, exists := os.LookupEnv("OVERRIDE_SES_ENDPOINT"); exists {
        return aws.Endpoint{
            PartitionID:   "aws",
            URL:           overrideEndpoint,
            SigningRegion: "eu-central-1",
        }, nil
    }

    return aws.Endpoint{}, &aws.EndpointNotFoundError{}
})

cfg, err := config.LoadDefaultConfig(context.TODO(),
    config.WithRegion("eu-central-1"),
    config.WithEndpointResolverWithOptions(customResolver),
)

client := sesv2.NewFromConfig(cfg)
```

## Manual testing

The SESv2 `SendEmail` endpoint is exposed at `POST http://localhost:8081/v2/email/outbound-emails` and expects the standard SESv2 JSON body:

```json
{
    "FromEmailAddress": "sender@example.com",
    "Destination": {
        "ToAddresses": ["recipient@example.com"],
        "CcAddresses": ["cc@example.com"],
        "BccAddresses": ["bcc@example.com"]
    },
    "ReplyToAddresses": ["reply-to@example.com"],
    "Content": {
        "Simple": {
            "Subject": {
                "Data": "Test email"
            },
            "Body": {
                "Text": {
                    "Data": "This is the message body in plain text format."
                },
                "Html": {
                    "Data": "<html><body><h1>Hello World!</h1></body></html>"
                }
            }
        }
    }
}
```

## :test_tube: Running Tests

```bash
go test ./internal/...
```

# :wave: Contributors

<a href="https://github.com/hatchddigital/aws-sesv2-mock/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=hatchddigital/aws-sesv2-mock" />
</a>

Maintained by [Hatchd Digital](https://hatchd.com.au). Originally created by [Askrella Software Agency](https://askrella.de)

# :warning: License

Distributed under the MIT License. See [LICENSE](LICENSE) for more information.
