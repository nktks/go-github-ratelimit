# go-github-ratelimit

[![Go Report Card](https://goreportcard.com/badge/github.com/gofri/go-github-ratelimit)](https://goreportcard.com/report/github.com/gofri/go-github-ratelimit)

Package `go-github-ratelimit` provides an http.RoundTripper implementation that handles [secondary rate limit](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api?apiVersion=2022-11-28#about-secondary-rate-limits) for the GitHub API.  
The RoundTripper waits for the secondary rate limit to finish in a blocking mode and then issues/retries requests.

`go-github-ratelimit` can be used with any HTTP client communicating with GitHub API.    
It is meant to complement [go-github](https://github.com/google/go-github), but there is no association between this repository and the go-github repository or Google.  
  
## Installation
```go get github.com/gofri/go-github-ratelimit```

## Usage Example (with go-github and [oauth2](golang.org/x/oauth2))
```go
import "github.com/google/go-github/github"
import "golang.org/x/oauth2"
import "github.com/gofri/go-github-ratelimit/github_ratelimit"

func main() {
  ctx := context.Background()
  ts := oauth2.StaticTokenSource(
    oauth2.Token{AccessToken: "Your Personal Access Token"},
  )
  tc := oauth2.NewClient(ctx, ts)
  rateLimiter, err := github_ratelimit.NewRateLimitWaiterClient(tc.Transport)
  if err != nil {
    panic(err)
  }
  client := github.NewClient(rateLimiter)

  // now use the client as you please
}
```

## Options
The RoundTripper accepts a set of options:
- User Context: provide a context.Context to pass to callbacks.
- Single Sleep Limit: limit the sleep time for a single rate limit.
- Total Sleep Limit: limit the accumulated sleep time for all rate limits.
  
The RoundTripper accepts a set of optional callbacks:
- On Limit Detected: callback for when a rate limit that requires sleeping is detected.
- On Single Limit Exceeded: callback for when a rate limit that exceeds the single sleep limit is detected.
- On Total Limit Exceeded: callback for when a rate limit that exceeds the total sleep limit is detected.

note: to detect secondary rate limits and take a custom action without sleeping, use SingleSleepLimit=0 and OnSingleLimitExceeded().

## Per-Request Options
Use WithOverrideConfig() to override the configuration for a specific request using a context.
Per-request overrides may be useful for special-cases of user requests,
as well as fine-grained policy control (e.g., for a sophisticated pagination mechanism).

## License
This package is distributed under the MIT license found in the LICENSE file.  
Contribution and feedback is welcome.
