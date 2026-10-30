# Pip Matrix Trading Platform

## Overview
Pip Matrix is a full-stack cryptocurrency and trading platform designed to allow users to manage investments across various asset classes, including crypto, forex, stocks, ETFs, and real estate. The platform aims to provide a comprehensive trading experience with features such as user authentication, cryptocurrency-based deposit/withdrawal management, demo trading, bot trading, copy trading, loan applications, and a referral system. Its ambition is to offer a unified solution for diverse investment needs, catering to both novice and experienced traders.

## User Preferences
Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend
- **Technology Stack**: Static HTML pages, Tailwind CSS for styling, Alpine.js for reactive UI.
- **Design Principles**: Multi-page application (MPA) architecture, dark mode by default with a custom indigo color palette, mobile-responsive design.
- **UI Components**: SweetAlert2 for notifications, Lucide and FontAwesome for iconography, Google Fonts (Inter).
- **State Management**: `localStorage` for theme persistence, Alpine.js for page-level reactivity.

### Backend
- **Framework**: Flask (Python).
- **API**: RESTful JSON API endpoints prefixed with `/api/`.
- **Authentication**: Session-based using Flask-Login with Werkzeug for password hashing.
- **File Serving**: Flask serves static HTML and assets.

### Database
- **ORM**: SQLAlchemy with Flask-SQLAlchemy.
- **Database**: PostgreSQL (configured via `DATABASE_URL`).
- **Connection Management**: `pool_pre_ping` for connection health checks.

### Data Models
Key entities include `User`, `Account` (live/demo balances), `Transaction` (deposits/withdrawals), `Investment`, `InvestmentWallet` (separate investment balances), `Trade`, `Loan`, `CopyTrading`, `BotTrading`, `Referral`, `SupportTicket`, `Notification`, `TradeRule` (admin-defined profit/loss rules), and `Subscription`.

### Payment Flow Business Logic (Critical)
The platform implements strict separation of payment flows:

1. **Trading Deposits** (affects main_balance):
   - User submits deposit with TXID → Transaction created as 'pending'
   - Admin approves → Account.balance (main_balance) increases
   - This is the ONLY flow that touches main_balance

2. **Premium Signal Subscriptions** (access rights only):
   - User pays for signal plan → Subscription created as 'pending'
   - Admin approves → Status changes to 'active', User.is_premium = True
   - NO balance credit - subscriptions are access rights, not funds

3. **Investment Plans** (separate investment wallets):
   - User invests → Investment created as 'pending' (NO balance deduction)
   - Admin approves → InvestmentWallet created with invested amount
   - Investment balances are tracked separately from trading balance

4. **Investment Profit Management** (admin-controlled):
   - Admin adds daily profit to investment wallet
   - Profit logged in `investment_profit_history` table
   - User notified and emailed about profit
   - Investment profits ONLY affect investment wallet balance, NEVER main_balance

Admin endpoints:
- `/api/admin/investments` - GET pending investments
- `/api/admin/investments/<id>/approve` - Approve investment
- `/api/admin/investments/<id>/reject` - Reject investment
- `/api/admin/subscriptions/<id>/approve` - Approve subscription
- `/api/admin/subscriptions/<id>/reject` - Reject subscription
- `/api/admin/investment-wallets` - GET all active investment wallets
- `/api/admin/investment-wallets/<id>/add-profit` - Add profit to investment wallet
- `/api/admin/investment-wallets/<id>/profit-history` - View profit history

User endpoints:
- `/api/investment-wallets` - GET user's investment wallets
- `/api/investment-wallets/<id>/profit-history` - View profit history

### Core Features & Implementations
- **Crypto Payment System**: Supports BTC, ETH, BNB, SOL, DOGE, USDT (TRC20), XRP. Features dynamic QR code generation, network display, TXID submission, and optional receipt upload. Admin approval for deposits and withdrawals.
- **Card Payment System**: Supports Visa, Mastercard and other cards via Flutterwave gateway. Features:
    - Country selection from 60+ supported countries
    - 3-step payment flow: Country → Amount → Gateway Payment
    - Gateway URL configurable via `CARD_PAYMENT_GATEWAY_URL` environment variable
    - Transaction reference submission and duplicate prevention
    - Minimum deposit of $50, integrated with admin panel for approval
    - API endpoints: `/api/card/countries`, `/api/card/gateway`, `/api/deposit/card`
- **Trading Systems**:
    - **Demo Trading**: Virtual balance, asset selection (crypto, stocks, forex), trade configuration (amount, leverage, order type), real-time trade summary, and account reset.
    - **Live Trading**: Real-money trading with balance validation, warning confirmations, and integration with the deposit system.
    - **Martingale Trading Chart**: Custom canvas-based chart on the dashboard with multiple asset types, timeframes, simulated real-time price updates, market sentiment, payout display, and trade timer. Trades automatically close upon timer expiry, calculating profit/loss based on entry/exit prices and admin-defined rules.
- **Admin Dashboard**: Secure, tab-based interface with admin authentication. Provides statistics, user management, new signup tracking, and comprehensive tools for managing deposits, withdrawals, subscriptions, and defining trade profit/loss rules.
- **Account History**: Consolidated view for all trades (demo and live) and a dedicated tab for live trade history with detailed insights.
- **File Uploads**: Handles payment receipts up to 16MB in the `uploads/` directory.

## External Dependencies

### Frontend Libraries (CDN)
-   Tailwind CSS
-   Alpine.js
-   SweetAlert2
-   Lucide Icons
-   FontAwesome
-   Google Fonts (Inter)

### Backend Python Packages
-   Flask
-   Flask-SQLAlchemy
-   Flask-Login
-   Werkzeug
-   qrcode

### Database
-   PostgreSQL

### External Services
-   **Resend**: Email service for transactional emails (verification, welcome, deposit confirmations, account funding, subscriptions). Configured via Replit connectors.
-   **Flutterwave**: Card payment gateway for deposits. Gateway URL stored in `CARD_PAYMENT_GATEWAY_URL` environment variable with default fallback.