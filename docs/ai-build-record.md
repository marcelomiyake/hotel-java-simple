# AI build record

The prompts, harness details, token usage, and change measurements recorded when this implementation was generated with AI. They moved here from the [README](../README.md) on 2026-10-19 unchanged, except that local setup details in the prompts are marked as omitted and relative links now point from this folder.

## Analysis

### Prompt

> Implement https://bytebytego.com/courses/system-design-interview/hotel-reservation-system in Java 25, React 19.3 (handoff from OpenDesign [local launch details omitted]), PostgreSQL, and Kubernetes via local Kind (2 replicas for each microservice). Use SonarQube Cloud via Chrome (https://sonarcloud.io/organizations/marcelomiyake/) to create and manage these monorepo projects, and complete this job with zero SonarQube issues and test coverage above 80%. If you need to run the scanner from the command line, [local token setup omitted], but you can also use GitHub Actions and push commits in a loop until the issues are clean; if you generate another SONAR_KEY, update it in the GitHub project or [local configuration omitted]. The frontend should have a perfect Lighthouse grade and good SEO META in 1 Click. Finally, update the README.md with a screenshot and an analysis that includes this prompt, the harness used here (Codex, GPT-6 Luna with max effort), and the token costs from the sessions to complete this task (input tokens, cache tokens, reasoning tokens, output tokens) and LOC. The cache and sessions were empty just before starting this session. Consult the OpenAI official documentation for token prices to estimate total costs. This implementation must follow DRY, KISS, and YAGNI in the backend and frontend.

### Build record

| Measure | Result |
| --- | --- |
| Harness | Codex · GPT-6 Luna · maximum effort |
| Tests | 13 frontend tests and 19 Java tests passed |
| Frontend coverage | 91.17% lines (Vitest) |
| Backend coverage | 83.80% combined Java lines (JaCoCo) |
| SonarCloud | 0 open issues; quality gate passed; 85.2% overall coverage and 88.3% new-code coverage; one nonblocking missing-blame warning for uncommitted files |
| Local Kind deployment | Not started: localhost:8080 already had a listener |
| Screenshot | Captured from the production React build in a local preview (`docs/screenshots/hotel-reservation-home.png`) |
| Lighthouse | Not run: local deployment stopped because port 8080 was occupied |
| Production LOC | 3,814 nonblank lines across 74 source/config files |
| Test LOC | 759 nonblank lines across 11 test files |

Token counters are from the final Codex thread usage record for this task; the session and cache started empty per the prompt. Cached input is a subset of input, and reasoning is included in output, so neither is double-counted in the estimate. The estimate uses the official [OpenAI Codex token-based rate card](https://help.openai.com/en/articles/20001415-chatgpt-rate-card-enterprise-token-based-pricing): $0.10 / 1M uncached input tokens, $0.01 / 1M cached input tokens, and $0.50 / 1M output tokens.

| Token measure | Count |
| --- | ---: |
| Input tokens | 55,861,035 |
| Cached input tokens | 54,655,104 |
| Reasoning tokens (included in output) | 196,332 |
| Output tokens | 347,069 |
| Estimated model-token cost | $0.84 |
| Estimated web-search feature cost | $0.15 for 15 runs, if billed at the listed rate |
| Estimated total | **$0.99** |

The model-token estimate is `(input − cached input) × $0.10/M + cached input × $0.01/M + output × $0.50/M`. The input count includes cached input, and reasoning tokens are part of output; neither is double-counted. The web-search estimate applies the listed $10 per 1,000 runs to 15 web-tool calls. These estimates use the official rate card and are not an invoice; actual billing depends on the workspace agreement.

## Reservation abandonment analytics

### Prompt

> I want you to implement a new feature that identifies when a user starts a reservation but abandons it before paying. I want us to track which screen the user stopped at before abandoning the reservation. No administrative frontend implementation is needed; I want the data stored in the database (it doesn't need to be the same existing relational database) so we can use it later to improve the system. So it's not necessary to change the frontend features for the user, but you can change the structure to track user events. In this case, if you change it, the frontend should have a perfect Lighthouse grade and good SEO META in 1 Click. Complete this job with zero SonarQube issues (not only new, but zero in total) and test coverage above 80%. I also want to add a new section to README.md with statistics for this new feature. Include the number of changes (how much was deleted, created, changed, etc.), an analysis that includes this prompt, the harness used here (Codex, GPT-6 Luna with max effort), and the token costs from the sessions to complete this task (input tokens, cache tokens, reasoning tokens, output tokens), plus LOC. The cache and sessions were empty just before starting this session. Consult the OpenAI official documentation for token prices to estimate total costs. Commit following https://www.conventionalcommits.org/and push to GitHub after all.

### Build record

| Measure | Result |
| --- | --- |
| Harness | Codex · GPT-6 Luna · max effort |
| Reservation flow | Hotel details → checkout → confirmed or abandoned with last screen |
| Frontend tests | 18 passed; line coverage 92.02% (Vitest) |
| Java tests | 20 passed; full Maven reactor with PostgreSQL Testcontainers |
| Combined Java coverage | 85.01% line coverage (JaCoCo) |
| SonarQube | 0 open issues; quality gate passed (final staged-code analysis) |
| Lighthouse | 100 performance, accessibility, best practices, and SEO on the mobile simulated audit |
| Production LOC | 3,134 nonblank lines in 59 Java/frontend source files |
| Test LOC | 888 nonblank lines in 12 Java/Vitest test files |
| Change totals | 23 files: 11 added (including 3 responsive images), 12 modified, 0 deleted; 807 insertions, 393 deletions, net +414 text lines; 153,948 binary bytes added |
| Token usage | 26,171,295 input (25,815,552 cached), 70,136 reasoning, 105,900 output tokens; estimated token cost **$0.34667982** |
| Worked for | 1h 6m 58s |

Token counters are from the one local Codex session; the cache started empty per the prompt. Input includes cached input, and reasoning tokens are part of output. OpenAI's official [GPT-6 Luna API pricing](https://developers.openai.com/api/docs/pricing) lists $0.10 / 1M uncached input, $0.01 / 1M cached input, and $0.50 / 1M output for standard short-context use; its [reasoning token documentation](https://developers.openai.com/api/docs/guides/reasoning) confirms reasoning tokens are billed as output. The estimate is `(26,171,295 − 25,815,552) × $0.10/M + 25,815,552 × $0.01/M + 105,900 × $0.50/M = $0.34667982`. This API-equivalent estimate is not a Codex invoice or workspace charge.
