---
title: Webhooks Explained - The Internet's Efficient Messengers
description: Dive into webhooks, how they differ from polling, their benefits for real-time updates, common use cases like payment notifications, and key considerations for implementation.
date: 2024-07-08
draft: false
slug: /pensieve/webhook
tags:
  - interview
  - SDE
---

Hey everyone! In the dynamic world of web applications, getting real-time updates is often crucial. Imagine waiting for a payment to process or a long-running task to complete. How does your application find out the result without constantly asking, "Are we there yet?" This is where **webhooks** come into play, offering a smart and efficient way for systems to communicate.

Let's explore what webhooks are, how they work, and why they've become a staple in modern system design, especially when interacting with third-party services.

## The Old Way: The Inefficiency of Constant Polling

Before we appreciate webhooks, let's consider the traditional alternative: **polling**.

Imagine your e-commerce website's order service needs to know the status of a payment handled by an external Payment Service Provider (PSP). With polling (specifically, short polling), your payment service would repeatedly send requests to the PSP asking, "Is the payment complete yet?". After several such requests, the PSP might finally respond with the status.

This approach has a couple of significant drawbacks:

* **Resource Intensive:** Constant polling consumes resources on your server (the payment service in our example) as it has to keep initiating requests. It also puts a load on the external service being polled.
* **Delayed Updates:** You only get updates when you poll. If an event happens right after a poll, you won't know until the next polling interval.
* **Potential Security Concerns:** Direct, frequent outbound connections for status checks might require more complex firewall rules or expose internal services more than necessary if not carefully managed.

## Enter Webhooks: "Don't Call Us, We'll Call You!"

Webhooks flip the script. A webhook is a mechanism that allows one system to notify another system in real-time via an HTTP callback when a specific event occurs. They are often referred to as **"reverse APIs"** or **"push APIs"** because, instead of your application making a request to the server for data, the server (or external service) makes an HTTP request to your application when it has something to report.

Here’s the general workflow:

1.  **Registration:** Your application provides a publicly accessible URL (the webhook endpoint) to the external service (e.g., the PSP). You essentially say, "When event X happens, please send the details to this URL".
2.  **Event Occurs:** The external service experiences the event (e.g., payment successfully processed, payment failed).
3.  **HTTP Callback:** The external service automatically sends an HTTP request (usually a POST request) to the registered webhook URL in your application, carrying a payload with information about the event.
4.  **Action:** Your application receives this incoming HTTP request, processes the payload, and takes appropriate action (e.g., update order status, notify the user).

This changes the programming paradigm. Your payment service doesn’t need to waste resources constantly polling for updates. It simply waits to be notified.

### An E-commerce Payment Example

Let's revisit our e-commerce scenario:

* Your `Order Service` sends a payment request to an external `Payment Service Provider (PSP)`.
* Instead of polling, your `Payment Service` has registered a webhook URL with the PSP.
* When the PSP completes the payment processing (successfully or not), it sends an HTTP POST request to your registered webhook URL with the transaction status.
* Your `Payment Service` receives this notification and updates the order status in your database.

## Important Considerations When Using Webhooks

While webhooks are powerful, implementing them robustly requires attention to a few details:

* **Handling Missed Callbacks:** What if the PSP never calls back due to a network issue on their end or a temporary glitch? It's wise to have a fallback mechanism. You could set up a housekeeping job (a scheduled task) to periodically check the status of payments that haven't received a webhook notification after a certain timeout.
* **Designing Your Webhook Endpoint:** You need to design a proper API endpoint in your application that the external service will call. This endpoint should be able to:
    * Securely receive incoming data.
    * Validate the data.
    * Process the event quickly (perhaps by putting it onto an internal queue for asynchronous processing to avoid keeping the external service waiting).
    * Return an appropriate HTTP status code (e.g., `200 OK` to acknowledge receipt).
* **Security:**
    * Since webhook URLs are publicly accessible, security is paramount. You'll need to set up proper rules in your API gateway or firewall to manage access.
    * **Verify Webhook Authenticity:** It's critical to ensure that incoming requests to your webhook URL are genuinely from the external service and not from a malicious actor. Common techniques include verifying signatures (e.g., HMAC shared secrets) or checking IP whitelists (though the provided documents focus on API gateway rules ).
* **Correct URL Registration:** Ensure the webhook URL you provide to the external service is correct and remains active. Any typos or changes without updating the external service will lead to missed notifications.
* **Idempotency:** Design your webhook handler to be idempotent. The external service might retry sending a webhook if it doesn't receive a timely acknowledgment or encounters an error. Your system should be able to handle the same event notification multiple times without causing duplicate processing or inconsistent state.

## Key Takeaways

* Webhooks provide an **event-driven** way for applications to receive real-time updates from external services via **HTTP callbacks**.
* They are more **resource-efficient** than traditional polling methods as they eliminate the need for constant status checks.
* Often called **"reverse APIs"** or **"push APIs"** because the external system initiates the communication to your application.
* Proper **security measures**, robust **error handling** (including missed callbacks), and **idempotent processing** are crucial for a successful webhook implementation.

Webhooks are a fantastic tool for building responsive, integrated applications. They enable services to communicate efficiently and react instantly to events as they happen.
