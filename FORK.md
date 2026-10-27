# General Inference fork

The `gi` branch carries integration fixes for `h2m-search` on top of upstream
`8bf8a75`.

- DuckDuckGo HTTP 202 responses are classified as challenges before testing
  success status, independently of the response body.
- Every search provider limits decoded response bodies to 2 MiB by default,
  checking both advertised lengths and streamed chunks before accumulation.
  Oversized bodies return the non-retryable `SearchError::ResponseTooLarge`.
- `DuckDuckGo::search_lite` lets hosts schedule the Lite fallback independently.
- `WikipediaBuilder::endpoint` supports mirrors and offline integration fixtures.
- Hosts can retain their own `reqwest::Client` and configure the body limit:

  ```rust
  let http = h2m_search::HttpConfig::with_client(client)
      .with_max_response_bytes(2 * 1024 * 1024);
  ```

These changes apply to search responses. They do not change `h2m`'s separate
page-scraping client or select different search providers.

Validation uses the repository's stable toolchain:

```sh
cargo fmt --all --check
cargo check --workspace --all-features
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace --all-features
cargo test --workspace --all-features
```

Regression tests use local HTTP fixtures, including chunked transfer without a
length header, exact size boundaries, JSON parse failures, provider-level limits,
and a 202 response whose body otherwise looks like valid search results.
