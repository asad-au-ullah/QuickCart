# QuickCart — Full Stack E-Commerce Platform

A full-featured e-commerce web application built with **Next.js**, featuring real-time authentication, cloud image storage, persistent cart management, and background job processing for order handling.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 16, Tailwind CSS |
| Authentication | Clerk (OAuth, email/password, user management) |
| Database | MongoDB (via Mongoose) |
| Background Jobs | Inngest (event-driven, batch processing) |
| Image Storage | Cloudinary |
| HTTP Client | Axios |
| Deployment | Vercel |

---

## Features

### Storefront
- Responsive homepage with hero slider, popular products, featured products, call-to-action, and email subscription sections
- Full product catalogue page with category browsing
- Product detail page with image gallery, description, ratings, and pricing

### Authentication & User Management
- Sign up / login via Google OAuth or email, powered by Clerk
- User profile dropdown with quick links to cart and order history
- Account management (profile picture, email) handled through Clerk's hosted UI

### Cart
- Add to cart and Buy Now flows from product pages
- Quantity controls with live updates
- Cart persisted to MongoDB — survives page reloads and sessions

### Checkout & Orders
- Delivery address management (add multiple addresses, select at checkout)
- Order placement triggers an Inngest event with **batch processing** (up to 5 concurrent orders) to manage database writes efficiently under load
- 2% tax calculated server-side at order creation
- Cart automatically cleared from the database on successful order placement

### Order History
- Users can view all past orders with product details, quantity, amount, and status
- Orders displayed in reverse chronological order

### Seller Dashboard (Role-Based)
- Seller role granted via Clerk public metadata (`role: seller`)
- **Add Product** — upload multiple product images (stored on Cloudinary), set name, description, category, price, and offer price
- **Product List** — view all products currently listed in the catalogue
- **Orders** — view all customer orders placed across the platform

---

## Architecture Highlights

**API Routes (Next.js App Router)**

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/inngest` | GET/POST | Inngest event handler (user sync, order creation) |
| `/api/user/data` | GET | Fetch authenticated user profile and cart |
| `/api/user/add-address` | POST | Save a new delivery address |
| `/api/user/get-address` | GET | Fetch all addresses for the current user |
| `/api/cart/update` | POST | Persist cart state to the database |
| `/api/cart/get` | GET | Retrieve cart state from the database |
| `/api/product/add` | POST | Add a new product (seller only) |
| `/api/product/list` | GET | Fetch all products for the storefront |
| `/api/product/seller-list` | GET | Fetch products for the seller dashboard |
| `/api/order/create` | POST | Place an order via Inngest event |
| `/api/order/list` | GET | Fetch orders for the current user |
| `/api/order/seller-orders` | GET | Fetch all orders for the seller dashboard |

**Clerk Webhooks via Inngest**
User lifecycle events (create, update, delete) are received from Clerk webhooks and processed by Inngest functions, keeping the MongoDB `users` collection in sync automatically.

**Batch Order Processing**
Orders are created through Inngest's batching feature. Events are grouped in batches of up to 5 and processed together using `insertMany`, reducing individual database write operations under concurrent load.

---

## Database Models

- **User** — synced from Clerk; stores profile info and cart items (as a key-value map of `productId: quantity`)
- **Product** — name, description, category, price, offer price, images array, seller reference, date
- **Address** — full name, phone, PIN code, area, city, state, linked to user
- **Order** — items array (product ref + quantity), amount, address ref, status, date, linked to user

---

## Environment Variables

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
MONGODB_URI=
INNGEST_EVENT_KEY=
INNGEST_SIGNING_KEY=
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=
```

---

## Getting Started

```bash
git clone https://github.com/your-username/quickcart.git
cd quickcart
npm install
# Add your environment variables to .env.local
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

To grant seller access to a user, add the following to their public metadata in the Clerk dashboard:

```json
{ "role": "seller" }
```
