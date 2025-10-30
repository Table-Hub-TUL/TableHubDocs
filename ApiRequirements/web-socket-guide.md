# TableHub Mobile Client: WebSocket Integration Guide

## Overview

The backend uses a hybrid system:

1.  **HTTP API:** Used for all "actions" or "commands," such as authenticating, fetching data, and *submitting* a table status update.
2.  **WebSocket (STOMP):** Used as a one-way channel for the *server to push* real-time updates to the client.

Clients **do not** send table updates *over* WebSocket. Clients send updates via an HTTP `POST` request, and the server broadcasts the change to all subscribed WebSocket clients.

---

## Step 1: Authentication (HTTP)

Before connecting to the WebSocket, you must get a JSON Web Token (JWT).

* **Endpoint:** `POST /auth/signin`
* **Body:**
    ```json
    {
      "username": "your_username",
      "password": "your_password"
    }
    ```
* **Response:**
    ```json
    {
      "token": "eyJh...A_57HioDE",
      "type": "Bearer",
      "username": "admin",
      "authorities": [...]
    }
    ```
* **Action:** Securely store the `token` value. This is required for all authenticated API calls and the WebSocket connection.

---

## Step 2: Connecting to the WebSocket (STOMP)

The server uses the **STOMP** protocol over WebSocket.

### Two-Part Authentication

The connection requires a **two-part authentication** scheme to be successful:

1.  **Handshake (URL Parameter):** The initial `ws://` connection is intercepted by `JwtHandshakeInterceptor`. This requires the token to be in the URL.
    * **URL:** `ws://YOUR_SERVER_HOST:8080/ws?token=YOUR_JWT_TOKEN`

2.  **STOMP Connect (Headers):** After the handshake, the STOMP client sends a `CONNECT` frame. This frame is intercepted by `AuthChannelInterceptor` and requires the token to be in the STOMP headers.
    * **STOMP Connect Headers:** `{"Authorization": "Bearer YOUR_JWT_TOKEN"}`

Your Kotlin STOMP client must be configured to provide *both* of these credentials at the correct time.

---

## Step 3: Subscribing to Topics

Once connected, you can subscribe to two different topics depending on the app's view.

### Topic 1: Aggregate Status (Map View)

This topic broadcasts *summarized* data for all restaurants. It's ideal for a map view where you only need to show total/free table counts.

* **Destination:** `/topic/restaurant-aggregates`
* **Payload:** `RestaurantStatusDto`
    ```json
    {
      "restaurantId": 1,
      "name": "Pierogi Paradise",
      "freeTableCount": 11,
      "totalTableCount": 12,
      "timestamp": "2025-10-30T10:30:00.123Z"
    }
    ```
* **Use Case:** Listen to this on your main map screen to show real-time availability on restaurant markers.

### Topic 2: Individual Table Status (Detail View)

This topic broadcasts *specific table changes* for a *single restaurant*. Clients should only subscribe to this when they are viewing a specific restaurant's details.

* **Destination:** `/topic/table-updates/{restaurantId}`
    * *Example:* `/topic/table-updates/1`
* **Payload:** `TableUpdateRequest`
    ```json
    {
      "restaurantId": 1,
      "sectionId": 1,
      "tableId": 1,
      "requestedStatus": "OCCUPIED"
    }
    ```
* **Use Case:** When a user enters a restaurant's detail page, subscribe to this topic to update the status of individual tables on the layout in real-time.

---

## Step 4: Triggering an Update (HTTP)

To report a table status change, you **must use the HTTP API**. The server will handle broadcasting the WebSocket messages.

* **Endpoint:** `POST /api/table/update-status`
* **Headers:** `{"Authorization": "Bearer YOUR_JWT_TOKEN"}`
* **Body:** `TableUpdateRequest`
    ```json
    {
      "restaurantId": 1,
      "sectionId": 1,
      "tableId": 1,
      "requestedStatus": "OCCUPIED"
    }
    ```
* **Server Response:** The server will:
    1.  Return `200 OK` to your HTTP request.
    2.  Send a `TableUpdateRequest` message to `/topic/table-updates/1`.
    3.  Send an updated `RestaurantStatusDto` message to `/topic/restaurant-aggregates`

---

## Full Mobile Workflow Example

1.  **App Start:**
    * `POST /auth/signin` to get JWT.
    * Connect to WebSocket (`ws://.../ws?token=...`) with STOMP headers (`Authorization: Bearer ...`).
    * Subscribe to `/topic/restaurant-aggregates`.
    * Populate map view (using `GET /api/restaurants/all` or `/api/restaurants`).
    * Listen for `RestaurantStatusDto` messages on the aggregate topic to update map markers.

2.  **User Enters Restaurant 1 Detail View:**
    * Fetch full layout: `GET /api/restaurants/1`.
    * Render the table layout from the HTTP response.
    * **Subscribe** to `/topic/table-updates/1`.
    * Listen for `TableUpdateRequest` messages to update individual tables.

3.  **User Reports Table 1 as "Occupied":**
    * Send `POST /api/table/update-status` with the update payload.
    * The app receives two messages almost instantly:
        1.  A `TableUpdateRequest` on `/topic/table-updates/1` (confirming the change).
        2.  A `RestaurantStatusDto` on `/topic/restaurant-aggregates` (updating the total count).

4.  **User Leaves Detail View:**
    * **Unsubscribe** from `/topic/table-updates/1` to save resources.
    * The subscription to `/topic/restaurant-aggregates` remains active.