# Per-language security footguns

Pull this up for the languages the diff actually changes. Each stack has sinks and traps a generic pass
misses. For each row: when the diff touches that surface, trace the source→sink path (SKILL.md step 2)
before you decide
it's a finding, and run it through the false-positive filter (step 4) before you report it.

## Python

| Trap | What to look for |
| ---- | ---------------- |
| Deserialization RCE | `pickle.loads`, `yaml.load` (without `SafeLoader`), `marshal`, `dill` on any untrusted bytes |
| Code eval | `eval`, `exec`, `compile` on input; `__import__`; f-string into `eval` |
| Command injection | `subprocess(..., shell=True)`, `os.system`, `os.popen` with interpolated input |
| Path traversal | `open`/`os.path.join` with user-controlled segments and no `os.path.realpath` containment check |
| SQL | f-string / `%`/`.format` into a query; `cursor.execute(f"...")` instead of param binding |
| SSRF | `requests.get(user_url)` with no allowlist; `urllib` to user host |
| Flask/Django | `render_template_string` on input (SSTI); `mark_safe` / `|safe` on tainted data; `DEBUG=True` |

## JavaScript / TypeScript (Node + browser)

| Trap | What to look for |
| ---- | ---------------- |
| Prototype pollution | recursive merge/clone/`Object.assign` over user JSON touching `__proto__` / `constructor` / `prototype` |
| Command injection | `child_process.exec`/`execSync` with interpolation (prefer `execFile` with an args array) |
| XSS | `dangerouslySetInnerHTML`, `innerHTML =`, `document.write`, `v-html` on tainted data |
| Code eval | `eval`, `new Function`, `setTimeout(string)`, `vm` with input |
| SQL/NoSQL | template-literal queries; Mongo operator injection (`$where`, user object spread into a filter) |
| SSRF | `fetch`/`axios` to a user-supplied URL with no host allowlist |
| Path traversal | `fs` calls with `req.params`/`req.query` segments; `path.join` without containment |
| Secrets | tokens in client bundles; `process.env` leaked into a response body |

## Java / JVM

| Trap | What to look for |
| ---- | ---------------- |
| Deserialization | `ObjectInputStream.readObject` on untrusted bytes; gadget-chain libs on the classpath |
| XXE | `DocumentBuilderFactory` / `SAXParser` / XMLInputFactory without `disallow-doctype-decl` / external entities disabled |
| Command/expr injection | `Runtime.exec` with interpolation; SpEL / OGNL / EL evaluating input |
| SQL | string-concatenated JDBC; JPA/HQL built from input instead of bound params |
| SSRF / path | `URL`/`HttpClient` to user host; `new File(userPath)` without canonical-path containment |
| Reflection | `Class.forName` / `Method.invoke` driven by input |

## Go

| Trap | What to look for |
| ---- | ---------------- |
| Template XSS | `text/template` where `html/template` was meant (no auto-escaping) |
| SSRF | `http.Get(userURL)` with no host validation; default client follows redirects |
| SQL | `fmt.Sprintf` into a query vs `db.Query(q, args...)` placeholders |
| Command injection | `exec.Command("sh", "-c", interpolated)` (prefer a fixed binary + args slice) |
| Path traversal | `filepath.Join` with input and no `filepath.Clean` + prefix check |
| Crypto | `math/rand` for tokens/secrets instead of `crypto/rand` |

## Ruby / Rails

| Trap | What to look for |
| ---- | ---------------- |
| Code eval | `eval`, `send`/`public_send` with a user-controlled method name, `constantize` on input |
| SQL | string-interpolated `where("...#{x}...")`; `find_by_sql` with interpolation |
| Mass assignment | missing strong-params; `permit!` |
| SSTI / XSS | `render inline:` on input; `raw` / `html_safe` on tainted data |
| Deserialization | `Marshal.load` / `YAML.load` on untrusted input |

## SQL (any host language)

- String concatenation / interpolation into a query is the default red flag; the fix is parameterized
  queries / prepared statements, not manual escaping.
- ORM escape hatches re-introduce the risk: `.raw()`, `.exec()`, `whereRaw`, `find_by_sql`, native-query
  APIs — treat anything that takes a query string as a sink.
- Second-order injection: input stored now, concatenated into a query later. Trace stored values that
  reach a query, not just request-time params.

## Cross-cutting (any language)

- **Secrets in the diff** — hardcoded keys/tokens/passwords, secrets logged, secrets in error responses
  or client bundles, secrets committed to config.
- **Authz, not just authn** — a new endpoint/handler that authenticates the user but never checks they
  may act on *this* resource (IDOR / missing object-level check).
- **Crypto** — home-rolled crypto, ECB mode, static IV/nonce, MD5/SHA1 for passwords (vs bcrypt/argon2),
  non-CSPRNG randomness for tokens, disabled TLS verification.
- **Dependency / config changes** — a bumped or added dependency, a loosened CORS/CSP, a disabled
  security middleware, a widened IAM/permission grant: review the config diff as a trust-boundary change.
