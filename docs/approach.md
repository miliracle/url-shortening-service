# Approach the Project: URL Shortener Service

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

**Question:** What is the traffic volume?

**Answer:** 500 million URLs are generated per month.

**Question:** How long is the shortened URL?

**Answer:** As short as possible.

**Question:** What characters are allowed in the shortened URL?

**Answer:** Shortened URL can be a combination of numbers (0-9) and characters (a-z, A-Z).

**Question:** Can shortened URLs be deleted or updated?

**Answer:** Yes, the owner could update and delete their URLs

**Question:** How do we identify the owner of a URL?

**Answer:** Let's generate a unique string key when the user creates a shortened link. This key can be used to update or delete the link.

### Basic Use Cases

1. **URL Redirecting:** Given a shorten URL, redirect to the original URL.
2. **URL Shortening:** Create shorten URL from the original URL.
3. **Update Short URL:** Update an existing shorten URL.
4. **Delete Short URL:** Delete an existing shorten URL.
5. **Get URL Statistics:** Retrieve statistics for the shorten URL (e.g., number of times accessed).
6. **High Availability, Scalability, and Fault Tolerance Considerations**

### Back of the Envelope Estimation

- **Traffic Estimation:**
  - **Write Operation:** 500 million URLs are generated per month.
  - **Write Operation per Second:** 500 million / (30 * 24 * 3600) = 200 URLs/s
  - **Read Operation:** With a read:write ratio of 100:1, read operations per second: 200 * 100 = 20,000 URLs/s

- **Storage Estimation:**
  - **Total Records over 5 Years:** With 500 million URLs generated per month, over 5 years we will have: 500 million * 12 * 5 = 30 billion URLs
  - **Storage Requirement:** Assuming each shortened URL requires 500 bytes of storage, the storage requirement over 5 years: 30 billion * 500 bytes = 15 TB

- **Bandwidth Estimation:**
  - **Incoming Data:** With 200 new URLs per second, incoming data: 200 * 500 bytes = 100 KB/s
  - **Outgoing Data:** With 20,000 read requests per second, outgoing data: 20,000 * 500 bytes = 10 MB/s

- **Memory Estimation:**
  - To improve system performance, we will cache frequently accessed shortened URLs.
  - Following the 80:20 rule, 20% of the shortened URLs generate 80% of the traffic.
  - With 20,000 requests per second, daily requests: 20,000 * 3600 * 24 = 1.7 billion requests/day
  - To cache 20% of these requests, memory required: 0.2 * 1.7 billion * 500 bytes = 170 GB

- **System Summary:**
  - 200 URLs are created per second
  - 20,000 requests per second
  - Incoming data: 100 KB/s
  - Outgoing data: 10 MB/s
  - Storage requirement over 5 years: 15 TB
  - Memory required for caching: 170 GB

## A: Architecture/High-level Design

![URL Shortener Architecture](https://assets.roadmap.sh/guest/url-shortener-architecture-u72mu.png)
Figure 1: Fullstack architecture, can be found at [https://roadmap.sh/projects/url-shortening-service](https://roadmap.sh/projects/url-shortening-service)

### Back-end

#### URL Redirecting

There are two ways to handle URL redirection: using HTTP status codes 301 and 302.

1. **301 Moved Permanently:**
   - Indicates the URL has been permanently moved.
   - Browsers cache the redirection and search engines update their indexes.
   - Not suitable for frequently changing URLs or tracking clicks.

2. **302 Found (Temporary Redirect):**
   - Indicates the URL is temporarily located at a different URL.
   - Browsers do not cache the redirection and search engines do not update their indexes.
   - Suitable for frequently changing URLs and tracking clicks.

Because we want to update the URL and track clicks, we will use the 302 status code for our URL redirection.

#### URL Shortening

//TODO

#### API Endpoint

The URL Shortener service will expose the following API endpoints to support the basic use cases:

1. **Create Short URL**
    - **Endpoint:** `/api/v1/shorten_urls`
    - **Method:** `POST`
    - **Description:** Create a shortened URL from the original URL. The response includes a unique key that can be used to modify the shortened URL and a shorten_code that will be used for redirection.
    - **Request Body:**

      ```json
      {
        "original_url": "https://www.example.com/very/long/url"
      }
      ```

    - **Response:**

      ```json
      {
        "key": "unique_key_for_modification",
        "shorten_code": "shortCode"
      }
      ```

2. **Get Shorten URL Details**
    - **Endpoint:** `/api/v1/shorten_urls/{shorten_code}?key={key}`
    - **Method:** `GET`
    - **Description:** Get the detail of the shorten URL, include the statistics information
    - **Path Parameter:**
      - `shorten_code`: The shortened URL identifier.
      - `key`: Authorize key
    - **Response:**

      ```json
      {
        "original_url": "https://www.example.com/very/long/url",
        "access_count": 999,
      }
      ```

3. **Update Shorten URL**
    - **Endpoint:** `/api/v1/shorten_urls/{shorten_code}`
    - **Method:** `PUT`
    - **Description:** Update the original URL associated with the shortened URL.
    - **Path Parameter:**
      - `shorten_code`: The shortened URL identifier.
    - **Request Body:**

      ```json
      {
        "key": "unique_key_for_modification",
        "original_url": "https://www.example.com/very/long/url",
      }
      ```

    - **Response:**

      ```json
      {
        "result": true
      }
      ```

4. **Delete Shorten URL**
    - **Endpoint:** `/api/v1/shorten_urls/{shorten_code}`
    - **Method:** `DELETE`
    - **Description:** Delete the shortened URL.
    - **Path Parameter:**
      - `shorten_code`: The shortened URL identifier.
    - **Request Body:**

      ```json
      {
        "key": "unique_key_for_modification"
      }
      ```

    - **Response:**

      ```json
      {
        "result": true
      }
      ```

5. **Redirect Shorten URL**
    - **Endpoint:** `/{shorten_code}`
    - **Method:** `GET`
    - **Description:** Redirect to the original URL.
    - **Path Parameter:**
      - `shorten_code`: The shortened URL identifier.
    - **Response:**
      - **Status Code:** `302 Found`
      - **Headers:**
        - `Location`: The original URL

### Front-end

![Front-end Architecture](fe_architech.png)

Figure 2: Front-end architecture diagram

#### Rendering appoarch

We will be using static page rendering

<details>
<summary>Static Page Rendering Approach</summary>

> Due to the simple use case of the front end and the lack of requirement for highly interactive features, we will be using a static page rendering approach.
>
> This approach will allow us to serve the necessary HTML, CSS, and JavaScript files directly from the server without the need for complex client-side rendering or dynamic content generation.
>
> Static page rendering is efficient and suitable for our needs, ensuring quick load times and ease of maintenance.
</details>

<details>
<summary>Other approach</summary>

> **Server-side rendering (SSR):**
>
> Rendering the HTML on the server side, which is the most traditional way. Best for static content that require SEO and does not require heavy user interaction. Websites like blogs, documentation sites, e-commerce websites are built using SSR.
>
> **Client-side rendering (CSR):**
>
> Rendering in the browser, by dynamically adding DOM elements into the page using JavaScript. Best for interactive content. Applications like dashboards, chat apps are built using CSR.

</details>

## D: Data Model

## I: Interface Definition (API)

## O: Optimizations and Deep Dive
