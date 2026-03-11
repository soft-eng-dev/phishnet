PhishNet

> Detecting phishing URLs before the damage is done, without relying on blocklists.

## The problem with blocklists

Blocklists only catch phishing URLs after they've been reported. But the damage is done before that happens — a phishing attack lives and dies in its first few hours. By the time a URL is reported, credentials have already been stolen.

PhishNet takes a different approach: instead of checking if a URL is on a known list, it looks at how the URL is structured and asks — does this *look* like a phishing URL?

## How it works

A Random Forest classifier trained on 235,000 labelled URLs, using 6 features engineered from the URL string itself:

| Feature | What it detects | Why it matters |
|---------|----------------|----------------|
| `is_https` | HTTP vs HTTPS | Phishing sites often skip HTTPS — only 49% of phishing URLs in our dataset used it vs 100% of legitimate ones |
| `url_length` | Length of the full URL | Legitimate sites have clean short URLs — phishing URLs are longer, hiding things or auto-generated |
| `digit_ratio` | Ratio of digits in the domain | Real brands are words humans chose (`netflix`, `google`) — machine-generated domains have random numbers like `f0519141` |
| `suspicious_tld` | TLD against a risk list | Attackers use cheap or free TLDs like `.gq`, `.tk`, `.ru` that cost nothing to register |
| `path_entropy` | Randomness of the URL path | Legitimate paths are readable (`/login`, `/search`) — phishing paths look randomly generated (`/aK93mZ/ryuie/`) |
| `brand_similarity` | Similarity to 1,000 known brands | Catches typosquatting — `netf1ix.com` scores 86% similarity to `netflix`, flagging it as suspicious |

## Results

```
              precision    recall  f1-score   support

  Legitimate       0.95      0.99      0.97     27035
    Phishing       0.99      0.93      0.96     20124

    accuracy                           0.96     47159
```

**96% accuracy, 0.93 phishing recall** — meaning 93% of phishing URLs are caught.

Recall matters more than precision here: it's better to block a legitimate URL occasionally than to let a phishing URL through and have someone's credentials stolen.

## What each feature contributes

```
is_https:         44.6% — strongest signal
url_length:       19.7%
digit_ratio:      14.8%
path_entropy:     13.9%
brand_similarity:  6.5% — catches typosquatting specifically
suspicious_tld:    0.5% — narrow but still contributes
```

## Known weaknesses

The model can be fooled by a URL that:
- Uses HTTPS (free SSL certificates are available to anyone)
- Is short
- Has no digits
- Uses a clean TLD
- Has a low-entropy path

Example: `https://www.netf1ix.com` passes most checks — only `brand_similarity` catches it. This is why diverse features matter — no single feature catches everything.

## Dataset

- **Source:** UCI ML Phishing Dataset
- **Size:** 235,795 URLs (134,850 legitimate, 100,945 phishing)
- **Brand reference:** Tranco top 1,000 domains

## Stack

`Python` `pandas` `scikit-learn` `fuzzywuzzy` `matplotlib` `Jupyter`