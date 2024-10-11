# Approach the Project

Following the [RADIO framework](https://www.greatfrontend.com/system-design/framework)
> Content inspired from: [System Design Interview An Insider's Guide by Alex Yu](https://www.amazon.com/System-Design-Interview-insiders-Second/dp/B08CMF2CQF)

*The requirements outlined in the roadmap.sh ([URL Shortening Service](https://roadmap.sh/projects/url-shortening-service)) are clear, but I would like to elaborate on them further.*

## Table of Contents

1. [R: Requirements Exploration](#r-requirements-exploration)
   - [Q&A for URL Shortener Service](#qa-for-url-shortener-service)
   - [Basic Use Cases](#basic-use-cases)
   - [Back of the Envelope Estimation](#back-of-the-envelope-estimation)
2. [A: Architecture/High-level Design](#a-architecturehigh-level-design)
3. [D: Data Model](#d-data-model)
4. [I: Interface Definition (API)](#i-interface-definition-api)
5. [O: Optimizations and Deep Dive](#o-optimizations-and-deep-dive)

## R: Requirements Exploration

### Q&A for URL Shortener Service

> **Question:** Can you give an example of how a URL shortener works?
>
> **Answer:** Assume URL [https://www.systeminterview.com/q=chatsystem&c=loggedin&v=v3&l=long](https://www.systeminterview.com/q=chatsystem&c=loggedin&v=v3&l=long) is the original URL. Your service creates an alias with shorter length: [https://tinyurl.com/y7keocwj](https://tinyurl.com/y7keocwj). If you click the alias, it redirects you to the original URL.

> **Question:** What is the traffic volume?
>
> **Answer:** 100 million URLs are generated per day.

> **Question:** How long is the shortened URL?
>
> **Answer:** As short as possible.

> **Question:** What characters are allowed in the shortened URL?
>
> **Answer:** Shortened URL can be a combination of numbers (0-9) and characters (a-z, A-Z).

> **Question:** Can shortened URLs be deleted or updated?
>
> **Answer:** Yes, the owner could update and delete their URLs

> **Question:** How do we identify the owner of a URL?
>
> **Answer:** Let's generate a unique string key when the user creates a shortened link. This key can be used to update or delete the link.

### Basic Use Cases

1. **URL Shortening:** Given a long URL, return a much shorter URL and the key.
2. **URL Redirecting:** Given a shorter URL, redirect to the original URL.
3. **Update Short URL:** Update an existing short URL with the key.
4. **Delete Short URL:** Delete an existing short URL with the key.
5. **Get URL Statistics:** Get statistics on the short URL (e.g., number of times accessed) with the key.
6. **High Availability, Scalability, and Fault Tolerance Considerations**

### Back of the Envelope Estimation

- **Write Operation:** 100 million URLs are generated per day.
- **Write Operation per Second:** 100 million / 24 / 3600 = 1160
- **Read Operation:** Assuming ratio of read operation to write operation is 10:1, read operation per second: 1160 * 10 = 11,600
- **Total Records over 10 Years:** Assuming the URL shortener service will run for 10 years, this means we must support 100 million *365* 10 = 365 billion records.
- **Storage Requirement:** Assume average URL length is 100 bytes. Storage requirement over 10 years: 365 billion * 100 bytes = 365 TB

## A: Architecture/High-level Design

![URL Shortener Architecture](https://assets.roadmap.sh/guest/url-shortener-architecture-u72mu.png)
> Figure 1: Fullstack architech, can be found at [https://roadmap.sh/projects/url-shortening-service](https://roadmap.sh/projects/url-shortening-service)

### Front-end

### Back-end

## D: Data Model

## I: Interface Definition (API)

## O: Optimizations and Deep Dive
