# XAuctionHouse

Auction house addon for XCore with fixed-price listings, live bidding auctions, and multi-currency support.

## Features

### Listing Types
- **Fixed Price** -- instant buy at seller's price
- **Auction** -- timed bidding with snipe protection, minimum increments, and automatic settlement

### Multi-Currency
- List and buy in any currency defined in XCore's economy system
- Default currency configurable (falls back to Vault primary)
- Prices displayed with correct currency symbol everywhere

### Bidding System
- Minimum bid increment (flat or percentage-based, configurable)
- **Configurable bid increment tiers** -- adjust +/- button values per starting bid magnitude in config
- Snipe protection -- extends auction if bid placed near end
- Automatic refund to outbid players
- Bid history tracked per auction item
- Maximum active auctions per player (configurable)

### Item Claim System
- **My Purchases** -- filter in My Items to view and claim bought items (full purchase history preserved)
- **Smart delivery** -- if buyer is online with inventory space, item is given directly; if inventory is full, item goes to My Purchases for claim
- **Auction wins** -- won items delivered directly if online, or stored for claim if offline/full inventory
- **Login notification** -- players notified of pending claims on join
- **Claimed items** -- stay visible in history with "Claimed" status indicator

### Marketplace
- **7 categories** -- All, Tools, Weapons, Blocks, Armors, Spawners, Other
- **Search** -- by item name, price range, material type
- **Sorting** -- newest/oldest, by category, by listing type (All / Buy Now / Auctions)
- **Favorites** -- bookmark items, dedicated favorites view, auto-purge obsolete entries, add/remove button in item lore updates in real-time
- **Shulker support** -- view contents before buying, blocked item validation inside shulkers

### Economy
- **Tax system** -- configurable rate, buyer or seller side
- **Sell limits** -- permission-based (`ah.limit.N`)
- **Pending payments** -- offline sellers paid automatically on next login
- **Seller notifications** -- sellers notified at login of all sales made while offline (configurable delay)
- **Price history** -- analytics table for sold item prices

### Visual Indicators
- **Not enough money** -- buy/bid button replaced with red "Not enough money" when player can't afford
- **Inventory full** -- buy button replaced with red "Inventory full" when no space (fixed-price only)
- **Translatable item names** -- item names displayed in player's language in chat messages and bossbar
- **Hoverable items** -- item names in chat show enchantments, durability, etc. on hover

### Integrations
- **Discord** -- webhooks for listings, sales, bids, auction wins, admin actions
- **PlaceholderAPI** -- sell count, sell limit, expired count, active auctions, active bids
- **Web API** -- REST endpoints for listings, sales, stats, player data
- **Cross-server sync** -- listings, purchases, and bids sync via XCore SyncManager

### GUIs (8 screens, fully YAML-configurable)
- **Auction House** -- main browse with sorting, filtering, pagination, favorites
- **My Items** -- for sale / expired / sold / my purchases / active bids tabs
- **Confirm** -- purchase confirmation with price, tax, and expiration display
- **Search** -- filtered results with same controls as main GUI
- **Bid Amount** -- select bid with +/- buttons, confirm
- **Bid Confirm** -- confirmation before placing bid
- **Player Auction** -- admin view of any player's items
- **Viewer** -- shulker box content preview

## Commands

| Command | Description | Permission |
|---------|-------------|------------|
| `/ah` | Open the auction house | - |
| `/ah sell <price> [quantity] [currency]` | List item at fixed price | - |
| `/ah auction <starting_bid> [duration] [currency]` | Start a timed auction | - |
| `/ah cancel` | View your listings | - |
| `/ah search <query> [min] [max] [material]` | Search items | - |
| `/ah favorite <item_uuid>` | Toggle favorite | - |
| `/ah favorites` | View favorited items | - |
| `/ah help` | Show help | - |
| `/ah reload` | Reload config, lang, GUIs | `ah.admin` |
| `/ah purge-expired` | Delete all expired items | `ah.admin` |
| `/ah purge-bought` | Delete all sold items | `ah.admin` |
| `/pah <player>` | View a player's items (admin) | `ah.admin` |

### Currency

If `[currency]` is omitted, the primary Vault currency from XCore is used (the one with `vault: true` in XCore's economy config).

### Duration

Auction duration format: `1h`, `6h`, `12h`, `1d`, `3d`, `7d`. Defaults to `auction.default-duration`. Must be between `auction.min-duration` and `auction.max-duration`.

## How Auctions Work

1. Seller runs `/ah auction <starting_bid>` with item in hand
2. Item appears in the auction house with an `[AUCTION]` tag
3. Buyers click the item to open the bid amount selector (+/- buttons)
4. Each bid must exceed the current bid by the configured minimum increment
5. Outbid players are automatically refunded
6. **Snipe protection**: bid placed near the end extends the auction
7. When auction ends:
   - **With bids**: highest bidder gets item (directly or via claim), seller receives payment (minus tax)
   - **No bids**: item returned to seller or marked as expired

## Permissions

| Permission | Description |
|------------|-------------|
| `ah.admin` | Admin commands (reload, purge, pah) |
| `ah.admin.remove` | Force-remove items (shift+left click in GUI) |
| `ah.limit.<N>` | Maximum active fixed-price listings |

## Configuration

```yaml
# Listing duration before expiration
duration: 3d

# Maximum price
maximum-price: 1000000000

# Default currency (empty = vault primary)
default-currency: ""

# Tax on sales
tax:
  enabled: false
  rate: 0.05          # 5%
  type: SELLER        # SELLER or BUYER

# Auction system
auction:
  enabled: true
  default-duration: 1d
  min-duration: 1h
  max-duration: 7d
  min-starting-bid: 1
  min-bid-increment: 1.0
  min-bid-increment-percent: 0    # 0 = use flat, >0 takes priority
  max-active-auctions: 5
  snipe-protection:
    enabled: true
    threshold: 5m
    extension: 5m
  # Configurable bid increment tiers
  bid-increments:
    - threshold: 100
      values: [1, 10, 50]
    - threshold: 1000
      values: [10, 100, 500]
    - threshold: 10000
      values: [100, 1000, 5000]
    - threshold: -1          # -1 = fallback
      values: [1000, 10000, 50000]
  announce-bids: false
  announce-wins: true

# Blocked items
blocked-materials: [ELYTRA]
blocked-names: [fuck]
blocked-anvil-names: [fuck]

# Announcements
announce-sale: true
announce-type: BOSSBAR    # CHAT, ACTION_BAR, TITLE, SUBTITLE, BOSSBAR

# Auto-return expired items on join
auto-return-expired: false

# Favorites
favorites:
  enabled: true

# Discord webhooks
discord-sales:
  enabled: false
  webhook-url: ""
discord-admin:
  enabled: false
  webhook-url: ""
```

## GUI Customization

All GUIs in `plugins/XCore/addons/XAuctionHouse/guis/`:

| File | Description |
|------|-------------|
| `auction.yml` | Main browse (TimeSort, CategorySort, ListingTypeFilter, BackPage, MyItems, NextPage, Favorites, Search) |
| `my_items.yml` | Player's items (TimeSort, CategorySort, Filter, BackPage, BackToAH, NextPage) |
| `confirm.yml` | Purchase confirmation (Confirm, Cancel) |
| `search.yml` | Search results (same as auction + ClearSearch) |
| `player_auction.yml` | Admin player view |
| `viewer.yml` | Shulker content viewer |
| `bid_amount.yml` | Bid selector (Plus1/10/100, Minus1/10/100, Display, Back, Confirm) |
| `bid_confirm.yml` | Bid confirmation |

Items support: `material`, `target-title`, `target-lore`, `target-button-on`/`off`, `permission`, `sound`, `custom_model_data_value`, `item_model_key`, `actions`.

## PlaceholderAPI

| Placeholder | Description |
|-------------|-------------|
| `%xauctionhouse_sell_limit%` | Max listing slots |
| `%xauctionhouse_sell_count%` | Active listings |
| `%xauctionhouse_expired_count%` | Expired items |
| `%xauctionhouse_active_auctions%` | Active auction count |
| `%xauctionhouse_active_bids%` | Items with player's highest bid |

## Web API

Base path: `/api/xauctionhouse` (requires XCore web dashboard)

| Endpoint | Description |
|----------|-------------|
| `GET /listings` | Active listings (includes currency, listing type, bid info) |
| `GET /sales` | Recent sales |
| `GET /stats` | Aggregate statistics |
| `GET /player/{name}` | Player's listings, sales, purchases |

## Database

| Table | Description |
|-------|-------------|
| `xauctionhouse` | Listings (currency, listing type, bid tracking) |
| `xauctionhouse_bids` | Bid history per auction |
| `xauctionhouse_favorites` | Player favorites |
| `xauctionhouse_price_history` | Sale price analytics |
