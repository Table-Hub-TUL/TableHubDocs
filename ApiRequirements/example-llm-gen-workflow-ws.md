# TableHub Real-Time Update Workflow

This document outlines the flow for updating restaurant table statuses and broadcasting those changes to clients using a hybrid WebSocket and message queue architecture.

---

## 1. Initial Setup (Client Starts) 📱

* **[Mobile Client] Login:** User authenticates via HTTP `POST /auth/signin`.
* **[Server Backend] Authentication:** `AuthController` validates credentials using `DaoAuthenticationProvider` / `UserDetailsServiceImpl` and issues a JWT via `JwtService`.
* **[Mobile Client] Store Token:** Client receives and stores the JWT.
* **[Mobile Client] WebSocket Connect:** Client initiates WebSocket connection to `/ws`, providing the JWT for authentication.
* **[Server Backend] WebSocket Auth:** `JwtHandshakeInterceptor` and `AuthChannelInterceptor` validate the token and establish an authenticated session.
* **[Mobile Client] Subscribe to Aggregates:** Client sends STOMP `SUBSCRIBE` frame to `/topic/restaurant-aggregates`.
* **[Server Backend] Register Subscription:** `SimpleBroker` notes the subscription.

---

## 2. Viewing Data 🗺️ / 🍽️

### A. Map View

* **[Mobile Client]** Listens for messages on `/topic/restaurant-aggregates`.
* **[Server Backend]** (When updates occur via Flow #3 below) `TableUpdateWorker` sends `RestaurantStatusDto` messages.
* **[Mobile Client]** Receives `RestaurantStatusDto` and updates the relevant restaurant marker/icon on the map with the new aggregate counts.

### B. Detailed Table View (Cold Start Handling)

* **[Mobile Client]** User taps on Restaurant `X`.
* **[Mobile Client]** **Immediately** sends HTTP `GET /api/restaurants/X` (authenticated with JWT).
* **[Server Backend]** `RestaurantController` calls `RestaurantDataService` to fetch details.
* **[Server Backend]** `RestaurantDataServiceImpl` queries the database for restaurant `X`, including all sections and current table statuses.
* **[Server Backend]** Returns `200 OK` with `RestaurantDetailedResponse` containing the full current state.
* **[Mobile Client]** Uses the REST response data to render the initial detailed view with correct table statuses.
* **[Mobile Client]** *Simultaneously*, sends STOMP `SUBSCRIBE` frame to `/topic/table-updates/X`.
* **[Mobile Client]** Listens for individual table update messages (payload: `TableUpdateRequest`) on `/topic/table-updates/X` and applies them to the view.
* **[Mobile Client]** When leaving the detailed view, sends STOMP `UNSUBSCRIBE` for `/topic/table-updates/X`.

---

## 3. Reporting a Table Status Update ✍️

* **[Mobile Client]** User marks a table as `AVAILABLE`/`OCCUPIED`.
* **[Mobile Client]** Sends authenticated HTTP `POST /api/table/update-status` with `TableUpdateRequest` body.
* **[Server Backend]** `JwtAuthTokenFilter` authenticates the request.
* **[Server Backend]** `TableStatusController` receives and calls `TableStatusService`.
* **[Server Backend] `TableStatusServiceImpl`:**
    1.  Finds table using efficient query (`RestaurantTableRepository.findByIdWithSectionAndRestaurant`).
    2.  Validates request data against fetched table data.
    3.  **(TODO):** Checks if points should be awarded based on status change.
    4.  Updates table status in the database (`RestaurantTableRepository.save`).
    5.  **(TODO):** If points awarded, saves `PointsAction` entity.
    6.  **Broadcasts *individual* update:** Sends `TableUpdateRequest` payload via `SimpMessagingTemplate` to `/topic/table-updates/{restaurantId}`.
    7.  **Queues *aggregate* job:** Sends `TableUpdateJob` (with `restaurantId`) via `RabbitTemplate` to RabbitMQ queue.
* **[Server Backend]** `TableStatusController` returns `200 OK` immediately.
* **[Mobile Client]** Shows instant success feedback to the reporting user.

---

## 4. Background Aggregate Processing ⚙️

* **[Server Backend] `TableUpdateWorker`:**
    1.  `@RabbitListener` receives `TableUpdateJob`, adds `restaurantId` to internal `Set`.
    2.  `@Scheduled` task runs periodically (e.g., every 0.75s).
    3.  Processes unique `restaurantId`s from the `Set`.
    4.  For each ID, queries `RestaurantTableRepository` for aggregate counts (`COUNT(*)`).
    5.  Creates `RestaurantStatusDto` with latest counts.
    6.  **Broadcasts *aggregate* update:** Sends `RestaurantStatusDto` via `SimpMessagingTemplate` to `/topic/restaurant-aggregates`.
* **[Mobile Client]** (Map View or Background) Receives `RestaurantStatusDto` and updates map markers.