### High-Level Plan

1. Functional Requirements[ Main Features ]
2. Non Functional Requirements (Platforms and Accessibility)
3. Component Architecture
4. Data Entities
5. Data API
6. Front-End Data Store
7. Infinite Scroll
8. Optimization
9. Accessibility

---

### Functional Requirements[Feature]

- Build an Infinite scrollable news feed.
- Stories appear based on:
  - User subscriptions (friends, group pages, etc.).
- Enable users to:
  - Share stories with images, links, videos, etc.
  - Post stories.
  - Attach comments, links, and media.

---

### Functional Requirements

- Feature must work on a wide range of devices.
- Must be accessible for users with disabilities.

---

### Component Architecture

![image.png](/public/nf-component.png)

- Basic Components:
  - **Story Component**:
    - Avatar
    - Title
    - Date of posting
    - Text and media
    - Control panel (Like, Comment, Share)
  - **Comment List**:
    - Displays comments below each story.
  - **Comment Input**:
    - Input field for adding comments.
- Structure:
  - News Feed = List of Stories
  - Dependency graph:
    - News Feed > Stories > Story Card > Comments (list and input).

---

### Data Entities

- **Story**:
  - `id`: Unique identifier.
  - `comments`: Array of comment IDs.
  - `media`: Array of media links.
  - `date`: Timestamp of creation.
  - `content`: Text content.
  - `origin`: Source of the story (e.g., Page, Group, User).
    - Fields: `type` (origin type), `name`.
- **Comment**:
  - `id`: Unique identifier.
  - `author`: User ID.
  - `media`: Media array.
  - `date`: Creation timestamp.
  - `content`: Text content.
- **Media**:
  - `type`: (e.g., Link, Video).
  - `url`: Media URL.

---

### Data API

- Endpoints:
  - **getPosts**:
    - Parameters: `api_key`, `user_id`,excludeComments?,cursor[pointer from where get data from],pageSize,minId.
    - Returns stories relevant to the user.
  - **createPost**:
    - Parameters: `api_key`, `user_id`,post_data.
  - **createComment**:
    - Parameters: `api_key`, `user_id`,comment_data.

### **API Protocols and Data Fetching Techniques**

### **API Calling Techniques**

1. **REST vs. GraphQL**:
   - **REST**: A traditional API approach using endpoints to fetch data. While **REST** is scalable and benefits from HTTP architecture features like caching, it can be less agile. Multiple endpoints are required to fetch different fields, which can be less efficient for multi-level data.
   - **GraphQL**: More flexible for handling multi-level data, as it allows clients to specify exactly which fields they need. However, it requires POST requests and doesn't natively handle caching, unlike REST, which supports built-in HTTP caching techniques.
   - **Choosing Between REST and GraphQL**: While both options are viable, **REST** is chosen here for scalability and the advantages it provides via HTTP caching.
2. **Data Fetching**:
   - After defining endpoints for REST API, data is fetched based on parameters. The fetched data is then pushed to the frontend store.

---

# **Frontend Datastore and Organization**

![image.png](/public/nf-fdo.png)

1. **Frontend Store Design**:
   - Frontend applications often need fast access to data, as they operate within browsers that run across various devices with limited resources. Efficient data organization is key.
   - The datastore should be **normalized** and **flattened** to optimize retrieval. Avoid multi-level structures that require extensive filtering.
   - **Flattened Structure**:
     - Data like **feeds** are stored in a map, where each feed is keyed by its ID. Associated data (e.g., users, media) is also keyed by ID, ensuring direct access without the need for nested structures.
     - **Example**: If accessing a **news feed**, the feed's ID is used to access the specific feed, and its associated comments or media are retrieved in a similar manner.
2. **Efficient Data Access**:
   - Data is fetched from the store by its ID, ensuring efficient access to the necessary pieces of information. This approach minimizes unnecessary computations and facilitates quicker rendering.

---

## **Handling Dynamic Data and Edge Cases**

1. **Dynamic Data Loading (Edge Case)**:
   - As new stories are added while scrolling (e.g., in a news feed), it's important to handle data fetching without overwhelming the system with unnecessary traffic.
   - **Challenges**: Constantly executing the same API request to fetch new posts can lead to traffic overhead. Efficient loading techniques are required to ensure that new stories appear without excessive delay or network traffic.
2. **Approaches for Dynamic Data Loading**:

   - **Long Polling**:
     - Involves making repeated requests at set intervals (e.g., every 200ms) to check for new data. While simple to implement, it has significant traffic overhead and increased latency, especially on mobile networks. Not ideal for production apps.
   - **WebSockets**:

     - Provides real-time, bidirectional communication, allowing data to be pushed instantly. However, WebSockets do not support HTTP/2, which can limit their scalability and efficiency, especially in production environments.
     - why http2 is imp

       HTTP/2 is important in this context because it significantly improves the performance and efficiency of web applications, particularly in scenarios involving dynamic data fetching and real-time updates. Here’s why HTTP/2 is key in this case:

       ### **1. Better Performance and Efficiency**

       - **Multiplexing**: HTTP/2 allows multiple requests and responses to be sent over a single connection concurrently. This reduces the need for multiple connections, improving the overall performance of the application, especially for real-time updates like news feeds.
       - **Header Compression**: HTTP/2 compresses headers, reducing the overhead of repetitive data transmission, which is particularly useful for APIs that make many small requests. This ensures faster data transmission and lower bandwidth usage.

       ### **2. Reduced Latency**

       - **Stream Prioritization**: HTTP/2 allows prioritization of streams, meaning more critical data can be delivered first, reducing the time it takes for the frontend to receive essential information. For example, in a news feed application, the most recent stories can be prioritized for faster loading.
       - **Less Overhead**: By using a single connection for multiple requests and responses, HTTP/2 reduces the need for handshakes between the client and server, lowering the latency of communication compared to older protocols like HTTP/1.1.

       ### **3. Server-Side Events (SSE) Compatibility**

       - **HTTP/2 Support for SSE**: Server-Side Events (SSE) work effectively over HTTP/2 because the protocol supports **persistent, multiplexed connections**. This means the client can stay connected to the server and receive updates without opening new connections each time, making it much more efficient than older techniques like long polling or WebSockets, which require separate connections for each update.

       ### **4. Scalability**

       - **Efficient Use of Resources**: HTTP/2 enables better resource utilization by reducing connection overhead and allowing multiple requests to be sent simultaneously. This is essential when handling large volumes of data or traffic, such as when fetching a stream of updates (e.g., news stories or social media posts).
       - **Load Balancing**: Since HTTP/2 supports multiplexing and prioritization, it’s easier to balance loads across multiple servers, making the system more resilient and capable of handling high traffic without sacrificing performance.

       ### **5. Real-Time Data Without Overhead**

       - **SSE over HTTP/2**: Using SSE over HTTP/2 enables the delivery of real-time updates in a very efficient manner. Unlike WebSockets, which don’t support HTTP/2 and have compatibility issues with HTTP/2’s advanced features, SSE leverages HTTP/2’s strengths like multiplexing, reduced overhead, and lower latency, making it more suitable for environments where efficiency and speed are critical.

   - **Server-Side Events (SSE)**:
     - A modern technique where the client subscribes to events on the server and receives updates in a binary format (not JSON). SSE works under HTTP/2, supports efficient data transfer, and minimizes overhead compared to long polling and WebSockets.
     - **Advantages**:
       - No traffic overhead like long polling.
       - Better load balancing compared to WebSockets.
       - Supported by HTTP/2, ensuring compatibility and improved performance.

3. **Choosing the Right Technique**:
   - **SSE** is chosen for efficiently handling dynamic data loading, as it provides the necessary real-time updates without the latency and overhead associated with WebSockets or long polling.

---

### **Implementation Steps and Final Thoughts**

1. **Final API Design**:
   - Add **SSE** as the endpoint for subscribing to real-time updates.
   - **Example Endpoint**: Replace typical `get` requests with `subscribe` requests for dynamic data fetching.
2. **Handling Edge Cases**:
   - By integrating SSE for real-time data, we effectively manage edge cases related to dynamic content loading, ensuring both performance and user experience are optimized.
3. **Conclusion**:
   - With **REST** for structured data access, a **normalized frontend store** for efficient data retrieval, and **SSE** for dynamic content updates, the system can handle large data loads and deliver a smooth user experience with minimal overhead.

---

---

---

# Infinite Scroll

![image.png](/public/nf-infinite.png)

- Key feature of Facebook News Feed.
- Implementation:
  - Dynamically load stories as the user scrolls down.
  - Maintain smooth performance to enhance user experience.

**What is Infinite Scroll?**

- Infinite scroll is a feature where a set of stories or content is continuously loaded as a user scrolls down. The new data appears when the user reaches the end of the current content.

**Why Use Infinite Scroll (e.g., Facebook News Feed)?**

- **Content Type**: Infinite scroll is often used for entertainment content where continuous interaction is desirable. It avoids pagination or "load more" buttons.
- **Efficiency**: Pagination or a "show more" button requires additional actions, while infinite scroll encourages the user to keep scrolling without interruptions.
- **User Engagement**: The aim is to keep users engaged and active on the website, without the need for additional actions.

**How is Infinite Scroll Implemented?**

- **Viewport and Stories**: The viewport refers to the visible area in the browser. It can display a varying number of stories (e.g., 10 stories per page).
- **Loading More Content**: As the user scrolls down, more content needs to be loaded when they reach the end of the current stories.

  **Intersection Observer**:

  - **Detecting Edge of Content**: The Intersection Observer API helps detect when the user reaches the end of the content (scrolls to the bottom).
  - **Nodes**: It uses two nodes:
    - **Top Sentinel**: Detects when the user scrolls up, triggering the re-display of previously shown data.
    - **Bottom Sentinel**: Detects when the user reaches the bottom of the page and triggers the loading of more data.

**Performance Considerations**

- **Large Data Volume**: As content grows (e.g., 500 stories), it’s important to manage performance and avoid loading too many DOM elements, especially on mobile devices.
- **Sliding Window Approach**: To prevent performance issues, a sliding window is used, which only shows a fixed number of stories in the viewport (e.g., 10 stories at a time).
- **Dynamic Story Replacement**: As the user scrolls down, the stories in the viewport are replaced with new ones, maintaining a constant number of DOM elements.

**Detailed Flow Example**

1. **Initial State**: The user sees the first 10 stories.
2. **Scroll Down**: As the user scrolls, the **intersection zone** triggers the loading of more data.
3. **Window Shift**: The visible window of stories shifts to show the next set (e.g., from 1-10 to 5-15), and only 10 stories are kept in the DOM at any time.
4. **Future Zone**: The data not currently in view is prepared for future loading.

**Top and Bottom Padding Nodes**

- **Purpose**: These nodes help create a smooth effect when loading more data.
  - When scrolling down, the **top padding** is increased, showing that more data is being loaded.
  - When scrolling up, the **top padding** is reduced, while the **bottom padding** increases to give the illusion that content is shifting smoothly.

**Edge Cases**

- The design should account for how to handle cases when the content reaches its end or when loading errors occur.

---

# Optimization

![image.png](/public/nf-optimization.png)

### Optimization Breakdown

1. **Optimization Aspects:**
   - **Network Performance**
   - **Rendering Performance**
   - **JavaScript Performance**
2. **Network Performance Optimization:**
   - **Optimize assets:** Compress resources (e.g., Brottle,GZIP files) to improve load times.
   - **Image Optimization:**
     - Serve images in modern formats like WebP if supported by the browser; fall back to PNG if not.
     - Use image services to send viewport details for optimized images. Caching optimized images on a CDN further improves performance.
     - Avoid serving content from distant locations (e.g., from Australia to Europe) by utilizing geo-location services.
   - **Use HTTP/2:**
     - HTTP/2 allows multiplexing, enabling parallel loading of hundreds of resources, which significantly improves performance.
   - **Resource Bundling:**
     - Split resources (e.g., news feed, header, vendor libraries, and analytics scripts) into bundles to avoid loading everything at once with http/2.
3. **Rendering Performance Optimization:**
   - **First Contentful Paint (FCP):**
     - Server-side rendering for certain pages (e.g., pre-rendering feeds) can reduce FCP time.
   - **Critical CSS and JavaScript Loading:**
     - In-line critical CSS and JavaScript to avoid blocking rendering. Use `async` or `defer` for scripts that do not impact initial rendering.
   - **CSS Optimization:**
     - Use efficient class naming strategies (e.g., BEM) and avoid deep CSS selectors to improve browser performance.
4. **JavaScript Performance Optimization:**
   - **Minimize Work:** Reduce the number of tasks and run them asynchronously to prevent blocking.
   - **Heavy Calculations:** Offload heavy computations to Web Workers to prevent blocking the main thread, improving interactivity.
   - **Caching Results:** Cache results of heavy calculations to speed up subsequent operations.
5. **Additional Optimization Techniques:**
   - **Lazy Loading for Images:** Only load images when they are within the user's viewport, reducing unnecessary resource loading.
   - **Placeholders:** Show placeholders or loaders while images or content are loading to reduce perceived load time for users.
6. **Progressive Web App (PWA) Features:**
   - **Offline Mode:** Enable offline capabilities using service workers and application cache to allow users to access cached resources even when not connected to the internet.

These strategies collectively optimize website performance, reducing load times, improving rendering efficiency, and enhancing user experience across different environments.

---

# Accessibility

**Accessibility Improvements**

- **Color Schemes:**
  - Provide alternative color schemes to accommodate users with color blindness or other visual impairments.
- **Screen Reader Support:**
  - Add appropriate ARIA (Accessible Rich Internet Applications) attributes to inputs, text areas, and other elements to make them compatible with screen readers.
- **Alt Attributes for Images:**
  - Ensure all images include descriptive `alt` attributes to improve accessibility.
- **Hotkeys:**
  - Introduce keyboard shortcuts for common actions such as:
    - Navigating to the next or previous story.
    - Posting a new story.
    - Scrolling up or down.
    - Returning to the main menu.
    - Accessing sharing options.

Originally From : 👉 [Front-End Engineer](https://www.youtube.com/@FrontEndEngineer)
