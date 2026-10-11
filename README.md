# CommUnique

[![App Store](https://img.shields.io/badge/App%20Store-shipped%20Sep%202026-0A0F2C.svg)](APP_STORE_LINK)
![tests](https://img.shields.io/badge/tests-~1%2C000%20across%2023%20suites-success.svg)
![privacy](https://img.shields.io/badge/privacy-anonymous%20by%20design-2EC4FF.svg)
[![site](https://img.shields.io/badge/site-communique.page-C8FF00.svg)](https://communique.page)

Take a photo of a public sign or a commercial display in Quebec. CommUnique
reads it, detects the language of each block, runs a deterministic 7-rule
engine, and returns a plain-language compliance report with the legal
references. It abstains rather than concluding when the read is uncertain.

This repository is the public showcase. The application source is private;
the architecture, the engineering decisions, the privacy design and the
numbers are all here.

```mermaid
flowchart LR
    P[Photo<br/>iOS, React Native / Expo] -->|EXIF stripped, encrypted client-side| U[Upload<br/>random device id, no account]
    U --> O[OCR<br/>Google Cloud Vision]
    O --> L[Per-block language detection]
    L --> E[Deterministic rule engine<br/>7 rules, each with a legal citation]
    E --> C{Confidence}
    C -->|high| R[Plain-language report<br/>findings + references]
    C -->|low| A[Abstain<br/>"not enough evidence to conclude"]
    E --> X[3-tier name lookup<br/>230,000 CIPO trademarks<br/>642,000 Quebec business names]
    R --> D[(Postgres<br/>90-day auto-deletion)]
```

## Numbers

| | |
|---|---|
| Automated tests | ~1,000 across 23 suites (Jest on the app, pytest on the backend) |
| Validation set | 3 regulator-published examples and 2 Quebec court rulings; the engine reproduces every outcome |
| Reference data indexed | 230,000 CIPO trademarks, 642,000 Quebec business names, 3-tier lookup (exact, normalized, fuzzy) |
| App Review | cleared on guidelines 5.1.1(iv) and 5.1.2(i), first submission |
| Downloads | _fill from App Store Connect_ |
| Reports generated | _fill from the backend counter_ |
| Precision on the validation set | _fill: correct findings / total findings_ |
| Median time photo to report | _fill from backend timing_ |

The last four rows are the ones that turn "I shipped an app" into a
number. They are read from App Store Connect and the backend's own counters
(request metadata only, see below) and updated monthly.

## Security and privacy by design

Quebec's Law 25 and Apple's review both ask the same question: what do you
keep, and why. The answer is as little as possible.

| Property | How |
|---|---|
| No account | A random device identifier is generated on first launch. There is no email, no login, no profile. |
| No location, no metadata | EXIF (GPS, device, timestamp) is stripped on the device before upload. |
| Encrypted in transit and before transit | TLS, and the image is encrypted client-side before it leaves the phone. |
| No PII at rest | The backend stores the report and a device id. Nothing in the report identifies a person. |
| Retention | Every image and report is deleted automatically after 90 days. There is no "keep forever". |
| Deterministic engine | No model decides compliance. Seven written rules do, each citing its legal basis, so every finding can be explained and contested. |
| Abstention | Low-confidence OCR produces "cannot conclude", never a guess. A wrong compliance verdict is worse than no verdict. |

## Stack

React Native with Expo and TypeScript on iOS. FastAPI and PostgreSQL on the
backend. Google Cloud Vision for OCR. Jest and pytest. Deployed on Google
Cloud.

## Why deterministic

The obvious design in 2026 is to hand the photo to a multimodal model and
ask "is this compliant". A model gives a fluent answer on every input,
including the ones where the honest answer is "cannot tell": partial reads,
mixed-language signs, brand names that are legally exempt. A rule engine
with a legal citation per rule and an explicit abstain path is slower to
write and much easier to defend, to a user, to a reviewer, and to a lawyer.
That is also why the validation set is regulator examples and court
rulings: the engine is held to the same standard a human reviewer would be.

## Links

- App Store: https://apps.apple.com/ca/app/communique/id6797602279 
- Site and how-it-works: [communique.page](https://communique.page)
- Author: [Jean-Cédrik Dorélas](https://jcedrik.com)
