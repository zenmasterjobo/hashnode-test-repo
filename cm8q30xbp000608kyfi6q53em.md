---
title: "Announcing the Square Go SDK"
datePublished: Wed Mar 26 2025 15:31:07 GMT+0000 (Coordinated Universal Time)
cuid: cm8q30xbp000608kyfi6q53em
slug: announcing-the-square-go-sdk

---


# It’s Time to Get ‘Go’ing
After much anticipation, and many asks from our community, we are very excited to announce Square now has a Go SDK! For Go SDK details, please visit our [website](https://developer.squareup.com/docs/sdks/go). If you are looking to get started quickly with the SDK, please check out this [quick-start guide](https://developer.squareup.com/docs/sdks/go/quick-start).

# The Go SDK in a Nutshell
If you are already familiar with our other SDKs, you will recognize many similarities between those and our Go SDK – however, our SDK was built to be idiomatic with Go, which will make the integration feel natural for your project.
## A Few Highlights
  - **Dependency Free** - This SDK is built completely on top of the standard library and fully compatible with `net/http` libraries.
  - **Auto-Pagination** - With the SDK, you can easily paginate through long responses. [Read more here](https://github.com/square/square-go-sdk?tab=readme-ov-file#automatic-pagination).
  - **Request Retries** - The SDK will handle request retries for you, as well as allow you to [alter](https://github.com/square/square-go-sdk?tab=readme-ov-file#retries) the retry strategy.

Our Go SDK is composed of many clients (based on our available APIs) which are all branched off of a general square.Client.

## SDK Usage
Install the Go SDK to your project with: 

`go get github.com/square/square-go-sdk`

You can then initialize the square.Client in your code like the following sample.
```go
import (
    "fmt"
    "context"

    square "github.com/square/square-go-sdk"
    squareclient "github.com/square/square-go-sdk/client"
    "github.com/square/square-go-sdk/option"
)

client := squareclient.NewClient(
    option.WithToken("<YOUR_ACCESS_TOKEN>"),
    // defaults to the production environment
    option.WithBaseURL("https://connect.squareupsandbox.com")
)
```

From here, our new variable `client` has access to the other API clients that compose this SDK. For example, requesting to see the locations for a given access token would like like this:

```go
response, err := client.Locations.List(context.TODO())

fmt.Println(*response.Locations[0].ID)
```

Check out our [API Explorer](https://developer.squareup.com/explorer/square) to generate code examples in Go for any of [our APIs](https://developer.squareup.com/reference/square) you are interested in using.

## What’s Next?
We are so excited to see what you build with the Go SDK, and we want to hear your feedback! Please submit feedback to our [forums](https://developer.squareup.com/forums/), [Discord](https://discord.com/invite/squaredev), or on the [Github repository's issues](https://github.com/square/square-go-sdk/issues). Happy coding!
