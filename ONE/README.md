# ONE — Point of Sale & Branch Management System

**A full web-based POS for a retail chain that sells ready-made products and drinks prepared on site.**
Arabic, right-to-left, works on desktop, tablet and phone, in light and dark mode.

![ONE demo: a cashier sale, then the manager's view](demo.gif)

▶ **[Watch the demo in full quality (MP4, 1 min)](demo.mp4)**

---

## The problem

A chain of about five branches sells packaged snacks and drinks made to order (coffee, tea, juices).
The owner needed to:

- sell fast with a barcode scanner, with no mistakes in the change;
- know the **real stock** of each branch, including the ingredients drinks are made from;
- know the **real profit** of every product and drink;
- stop manipulation: discounts, hospitality, credit sales and stock changes need approval, and every sensitive
  action is logged;
- follow every branch live from one place, **including from a phone**.

## The solution

One central web app with three roles, each seeing only what it needs:

| Role | What they do |
|---|---|
| **Cashier** | Sells by barcode or touch, adds drink notes (“light sugar”, “take-away”), gets the change automatically, prints the receipt and the drinks order for the bar, opens and closes their shift. |
| **Supervisor** | Everything a cashier does, plus returns, cancellations, discounts within each item's limit, approving hospitality and credit sales with a PIN, purchase invoices, supplier payments, the cash book and stocktakes for their branch. |
| **Manager** | All branches from one screen: live overview, approvals, catalog and prices, recipes, employees and users, suppliers and customers, and the final approval of purchases and stocktakes. |

---

## Screens

### Point of sale

| Cart, with drink notes and quick-cash buttons | Paid: receipt and drinks order printed |
|---|---|
| ![POS cart](images/03-pos-cart.png) | ![Sale complete](images/04-pos-paid.png) |

### Manager

| Live overview of every branch | Approvals and notifications (live) |
|---|---|
| ![Overview](images/02-overview.png) | ![Approvals](images/08-approvals.png) |

| Purchase invoice (supervisor enters, manager approves) | Supplier accounts |
|---|---|
| ![Purchase invoice](images/09-purchase.png) | ![Supplier accounts](images/13-supplier-accounts.png) |

| Catalog: products, drinks and their recipes | Cash book |
|---|---|
| ![Catalog](images/11-catalog.png) | ![Cash book](images/12-cash-book.png) |

### Stock and stocktake

| Stock per branch: quantities, cost and sale value | Cashier shifts |
|---|---|
| ![Stock](images/05-stock.png) | ![Shift board](images/10-shift-board.png) |

| Blind count (the system quantity is hidden while counting) | Review the differences before the manager approves |
|---|---|
| ![Stocktake count](images/06-stocktake-count.png) | ![Stocktake review](images/07-stocktake-review.png) |

### Dark mode

| Overview | Point of sale | Stock |
|---|---|---|
| ![Dark overview](images/20-dark-overview.png) | ![Dark POS](images/21-dark-pos.png) | ![Dark stock](images/22-dark-stock.png) |

### On a phone

| Manager overview | Approvals | POS | Cart | Stock |
|---|---|---|---|---|
| ![Phone overview](images/30-phone-overview.png) | ![Phone approvals](images/31-phone-approvals.png) | ![Phone POS](images/32-phone-pos-items.png) | ![Phone cart](images/33-phone-pos-cart.png) | ![Phone stock](images/34-phone-stock.png) |

---

## Features

**Selling**
- Barcode scanning (USB or serial scanners), touch grid with categories, search by name.
- Cash sales with automatic change; credit sales to organizations with a PIN approval and later collections.
- Hospitality invoices: recorded, costed and approved.
- Returns (back to stock or to waste), cancellations, invoice lookup, price-check screen for customers.
- Cashier shifts: opening cash, closing count, the difference recorded.

**Stock and purchasing**
- One stock per branch, kept as an immutable ledger of movements: every sale, return, purchase and count.
- Drinks are sold through **recipes**: selling a latte consumes coffee, milk and a cup.
- Purchase invoices: the supervisor enters, the manager approves, then stock and supplier balance update.
- Supplier accounts and payments; low-stock alerts.
- **Stocktake**: a blind count with the scanner, a review of the differences with their cost, and approval by
  the manager with a PIN. **Sales in the branch stop automatically while the count is open**, so the numbers
  are exact.

**Control**
- Role permissions, and PIN approval for every sensitive action.
- An immutable audit log.
- Discount limits per item, set as a percent of the margin.
- Live notifications: approval requests arrive on the manager's screen and phone instantly.

**Printing**
- A small print agent on each cashier PC prints receipts, drinks orders for the bar (no prices, as many copies as needed)
  and barcode labels directly on thermal printers, with no browser print dialog.
- Arabic text is rendered correctly on any thermal printer.

---

## How it is built

```mermaid
flowchart LR
  subgraph Branch PC / phone
    W[React web app<br/>Arabic RTL · responsive · dark mode]
    A[Print agent<br/>thermal printers · scanners]
  end
  W -- REST / JSON --> D[Django REST API]
  W -- WebSocket --> C[Django Channels]
  W -- localhost --> A
  D --> P[(PostgreSQL<br/>rules enforced in the database)]
  P -- LISTEN / NOTIFY --> L[Notification listener] --> R[(Redis)] --> C
```

| Layer | Technology |
|---|---|
| Web app | React 19, TypeScript, Vite, Tailwind CSS 4, React Router |
| API | Django 6, Django REST Framework, OpenAPI 3 contract |
| Live updates | PostgreSQL LISTEN/NOTIFY → Django Channels (WebSockets) over Redis |
| Database | PostgreSQL: 42 tables, business rules in functions and triggers |
| Printing | Python agent: ESC/POS, Arabic text drawn as an image |
| Deployment | Docker (database and Redis) |

### Engineering highlights

- **Money and stock rules live in the database.** Stock can't go negative, the ledger can't be edited, and a
  stocktake freezes the branch. These are enforced by PostgreSQL triggers and functions, so no screen, API
  call or bug can get around them.
- **Safe when many people act at once.** Two cashiers selling the last item, two managers approving the same
  invoice, and a count starting while sales are running are all handled with row and advisory locks, and each
  case has a dedicated race test.
- **API contract first.** The frontend and backend share an OpenAPI contract with real recorded examples.
  Every endpoint is tested against it.
- **Built-in demo mode.** The web app can run entirely in the browser with demo data (as in the screenshots),
  so it can be shown to anyone without a server.

---
