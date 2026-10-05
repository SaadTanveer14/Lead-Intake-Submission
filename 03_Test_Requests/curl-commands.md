# Smart Lead Intake API – curl Test Commands

Endpoint: `POST https://3zhkbl.buildship.run/leadIntake`

Each command prints the JSON response, then `HTTP 200` or `HTTP 400`. Ship the workflow in BuildShip before running.

| # | Test | Expected |
|---|------|----------|
| 1 | Clear buying intent | 200, priority hot, Slack + email alert, row saved |
| 2 | General question, no budget or timeline | 200, priority warm or cold, row saved, no alert |
| 3a | Missing email | 400, errors: [`email is required`], nothing saved |
| 3b | Invalid email format | 400, errors: [`email is not valid`], nothing saved |
| 4 | Message shorter than 10 characters | 400, errors: [`message must be at least 10 characters`], nothing saved |
| 4b | Several problems at once | 400, three errors listed |
| 6 | Website redesign, no timeline | 200, likely warm, web-development |
| 7 | Mobile app with deadline and budget | 200, likely hot, mobile-app |
| 8 | Job application | 200, likely cold, other |
| 9 | Automation interest | 200, likely warm, automation |
| 10 | SEO sales pitch / spam | 200, likely cold, other |
| 11 | UI/UX audit, urgent | 200, likely hot, ui-ux-design |

**AI failure test:** point Classify Lead's *Gemini API Key* at a wrong value, ship, re-run test 2 → expect `200` with `"priority":"unscored"` and the row still saved. Then restore `GEMINI_API_KEY` and ship again.

## macOS / Linux / Git Bash

**1. Clear buying intent**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Sara Ahmed","email":"sara@acme.com","company":"Acme Ltd","message":"We need a chatbot for our store, budget ready, want to start this month.","source":"website-contact-form"}'
```

**2. General question, no budget or timeline**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Ali Khan","email":"ali@example.com","company":"","message":"Hi, just curious what kind of services you offer?","source":"website-contact-form"}'
```

**3a. Missing email**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Bob Smith","company":"X Corp","message":"We would like a new website for our firm.","source":"website-contact-form"}'
```

**3b. Invalid email format**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Bob Smith","email":"bob@","company":"X Corp","message":"We would like a new website for our firm.","source":"website-contact-form"}'
```

**4. Message shorter than 10 characters**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Bob Smith","email":"bob@xcorp.com","company":"X Corp","message":"hi","source":"website-contact-form"}'
```

**4b. Several problems at once**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"B","email":"not-an-email","message":"short"}'
```

**6. Website redesign, no timeline**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Hina Raza","email":"hina@bloomflorist.pk","company":"Bloom Florist","message":"Our website looks outdated and we would like a redesign with online ordering. Can you share your process?","source":"website-contact-form"}'
```

**7. Mobile app with deadline and budget**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Usman Tariq","email":"usman@fitlife.io","company":"FitLife","message":"We have a 15k USD budget for an iOS and Android fitness app and need an MVP before December. Can we book a call this week?","source":"referral"}'
```

**8. Job application**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Ayesha Noor","email":"ayesha.noor@gmail.com","company":"","message":"Hello, I am a junior React developer and would love to apply for an internship at your studio.","source":"website-contact-form"}'
```

**9. Automation interest**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Daniel Brooks","email":"daniel@ledgerly.co","company":"Ledgerly","message":"We spend hours copying invoices into spreadsheets. Is this something you could automate for us?","source":"linkedin"}'
```

**10. SEO sales pitch / spam**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Mark Lee","email":"mark@seoboost.biz","company":"SEO Boost","message":"We can get your website to page one of Google in 30 days. Reply for our special offer!","source":"website-contact-form"}'
```

**11. UI/UX audit, urgent**

```bash
curl -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d '{"name":"Fatima Sheikh","email":"FATIMA@Payzen.com ","company":" PayZen ","message":"  Our checkout conversion dropped. We have budget approved for a UX audit and redesign and want to start ASAP.  ","source":"website-contact-form"}'
```

## Windows CMD

Uses `curl.exe` with escaped quotes. In PowerShell, type `cmd` first.

**1. Clear buying intent**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Sara Ahmed\",\"email\":\"sara@acme.com\",\"company\":\"Acme Ltd\",\"message\":\"We need a chatbot for our store, budget ready, want to start this month.\",\"source\":\"website-contact-form\"}"
```

**2. General question, no budget or timeline**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Ali Khan\",\"email\":\"ali@example.com\",\"company\":\"\",\"message\":\"Hi, just curious what kind of services you offer?\",\"source\":\"website-contact-form\"}"
```

**3a. Missing email**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Bob Smith\",\"company\":\"X Corp\",\"message\":\"We would like a new website for our firm.\",\"source\":\"website-contact-form\"}"
```

**3b. Invalid email format**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Bob Smith\",\"email\":\"bob@\",\"company\":\"X Corp\",\"message\":\"We would like a new website for our firm.\",\"source\":\"website-contact-form\"}"
```

**4. Message shorter than 10 characters**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Bob Smith\",\"email\":\"bob@xcorp.com\",\"company\":\"X Corp\",\"message\":\"hi\",\"source\":\"website-contact-form\"}"
```

**4b. Several problems at once**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"B\",\"email\":\"not-an-email\",\"message\":\"short\"}"
```

**6. Website redesign, no timeline**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Hina Raza\",\"email\":\"hina@bloomflorist.pk\",\"company\":\"Bloom Florist\",\"message\":\"Our website looks outdated and we would like a redesign with online ordering. Can you share your process?\",\"source\":\"website-contact-form\"}"
```

**7. Mobile app with deadline and budget**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Usman Tariq\",\"email\":\"usman@fitlife.io\",\"company\":\"FitLife\",\"message\":\"We have a 15k USD budget for an iOS and Android fitness app and need an MVP before December. Can we book a call this week?\",\"source\":\"referral\"}"
```

**8. Job application**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Ayesha Noor\",\"email\":\"ayesha.noor@gmail.com\",\"company\":\"\",\"message\":\"Hello, I am a junior React developer and would love to apply for an internship at your studio.\",\"source\":\"website-contact-form\"}"
```

**9. Automation interest**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Daniel Brooks\",\"email\":\"daniel@ledgerly.co\",\"company\":\"Ledgerly\",\"message\":\"We spend hours copying invoices into spreadsheets. Is this something you could automate for us?\",\"source\":\"linkedin\"}"
```

**10. SEO sales pitch / spam**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Mark Lee\",\"email\":\"mark@seoboost.biz\",\"company\":\"SEO Boost\",\"message\":\"We can get your website to page one of Google in 30 days. Reply for our special offer!\",\"source\":\"website-contact-form\"}"
```

**11. UI/UX audit, urgent**

```
curl.exe -s -w "\nHTTP %{http_code}\n" -X POST https://3zhkbl.buildship.run/leadIntake -H "Content-Type: application/json" -d "{\"name\":\"Fatima Sheikh\",\"email\":\"FATIMA@Payzen.com \",\"company\":\" PayZen \",\"message\":\"  Our checkout conversion dropped. We have budget approved for a UX audit and redesign and want to start ASAP.  \",\"source\":\"website-contact-form\"}"
```
