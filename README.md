# XAuctionHouse

An auction house for XCore: fixed-price listings, live auctions, several currencies.

## Features

### Listing types

Fixed price, bought instantly. Auctions, running for a set time with snipe protection, a minimum
increment and automatic settlement.

### Currencies

Any currency defined in XCore's economy. The default one is configurable and falls back to the
primary Vault currency. Prices are shown with the right symbol.

### Bidding

Minimum increment as a flat amount or a percentage. The `+` and `-` buttons adjust by amounts
configured in tiers, following the size of the current bid. A bid near the end extends the auction,
an outbid player is refunded, and each item keeps its bid history. The number of auctions per player
is configurable.

### Claiming

A purchase is delivered directly when the buyer is online with room, otherwise it waits in My
Purchases, as does an auction win. Players are told on join that something is waiting. A claimed
item stays in the history marked as claimed.

### Housekeeping

An expired listing nobody reclaims used to stay in memory and be reloaded at every start, for
good. `purge.expired-days` hands it back to its seller through XCore's mailbox, so they get it on
their next login, and removes the row. `purge.history-days` sets how long a
fully settled sale is kept as history. Either window is disabled by setting it to 0.

### Browsing

Eleven categories: All, Equipment, Blocks, Redstone, Resources, Food, Potions, Enchanting,
Decoration, Spawners, Other, plus Favorites. Categories are computed from the item and never stored,
so changing the set needs no migration.

Search by name, price range or material. Sort by newest or oldest, by category or by listing type.
Favorites have their own view and drop obsolete entries. A shulker box can be opened before buying,
and the blocked-item rules apply to its contents.

### Money

Tax rate and side are configurable and apply to fixed-price sales and auctions. Under `BUYER` the
tax is added to the bid when it is placed, under `SELLER` it comes off the payout.

An item changes hands only once the money is collected: a refused withdrawal aborts the purchase.
Vault calls run on the server thread.

Sell limits come from `ah.limit.N`. An offline seller is paid on their next login and told what
sold. Every sale feeds the price history table.

`/ah serverlist <price> [infinite] [currency]` creates a server listing: the buyer pays, nobody is
paid. `single` is consumed on purchase, `infinite` stays in stock. They never expire and show the
configurable `Server` seller name.

### Market prices

`/ah price [material]` gives the average, lowest, highest and last price over the configured window,
seven days by default. Hold an item and type `/ah price` to price it. The dashboard has a Market
page ranking materials by volume.

### Standing alerts

`/ah alert add <material> [max price]` notifies when a matching item is listed, offline included.
Alerts are checked when a listing is created. `/ah alert`, `/ah alert remove <material>`,
`/ah alert clear`.

### Quick buying

`/ah buy <material> [max price]` takes the cheapest matching listing through the usual confirmation.

### Commission per rank

`ah.tax.<name>` gives a rank its own tax rate. The lowest rate a player holds wins.

### Restricted materials

`restricted-materials` with `ah.sell.<material>` limits who can list a material.

### Moderation

`/ah ban <player> <duration|def> [reason]` and `/ah unban <player>`, working on offline players
(`def` = permanent). Selling, buying, bidding and opening the auction house are four separate
switches. A banned player's listings are removed on ban and returned if they are online.
`ah.admin` players are exempt by default. Bans sync across servers.

### What players see

The buy or bid button becomes a red "Not enough money" or "Inventory full". Item names are shown in
the player's own language in chat and on the boss bar, and hovering one shows its enchantments and
durability.

### Integrations

Discord webhooks for listings, sales, bids, auction wins and admin actions, sent through XCore so a
busy market delays notifications rather than losing them. PlaceholderAPI. A dashboard with listings,
sales, statistics, player lookup and a Bans page. Listings, purchases and bids sync through XCore's
SyncManager.

### Menus

Eight screens, configurable in YAML: the main browse, My Items with its tabs, purchase
confirmation, search results, the bid selector, bid confirmation, the admin view of a player, and
the shulker viewer. The four browsing screens share one implementation, load their listings off the
main thread on every open and every page turn, and redraw in place when a sort or a filter changes.

## Commands

| Command | Description | Permission |
|---------|-------------|------------|
| `/ah` | Open the auction house | - |
| `/ah sell <price> [quantity] [currency]` | List the held item at a fixed price | - |
| `/ah auction <starting_bid> [duration] [currency]` | Start an auction | - |
| `/ah cancel` | Your listings | - |
| `/ah search <query> [min] [max] [material]` | Search | - |
| `/ah price [material]` | Average, lowest, highest and last sale price | - |
| `/ah stats [player]` | What you have listed, sold and bought | `ah.admin` for another player |
| `/ah buy <material> [max price]` | Buy the cheapest match | - |
| `/ah alert add\|remove\|clear` | Standing alerts | - |
| `/ah favorite <item_uuid>` | Toggle a favorite | - |
| `/ah favorites` | Your favorites | - |
| `/ah help` | Help | - |
| `/ah reload` | Reload config, language files and menus | `ah.admin` |
| `/ah purge-expired` | Delete expired items | `ah.admin` |
| `/ah purge-bought` | Delete sold items | `ah.admin` |
| `/ah serverlist <price> [infinite] [currency]` | Create a server listing | `ah.admin` |
| `/ah ban <player> <duration\|def> [reason]` | Ban from the auction house | `ah.admin` |
| `/ah unban <player>` | Lift the ban | `ah.admin` |
| `/pah <player>` | View a player's items | `ah.admin` |

Extra command names go under `command-aliases`. `command-aliases.ah: [hdv, auctionhouse]` adds
`/hdv` and `/auctionhouse`. `ah` and `pah` always work. Aliases ignore case and cover every
subcommand.

Without `[currency]`, the primary Vault currency from XCore is used.

Auction durations: `1h`, `6h`, `12h`, `1d`, `3d`, `7d`, between `auction.min-duration` and
`auction.max-duration`, defaulting to `auction.default-duration`.

## How an auction runs

1. `/ah auction <starting_bid>` holding the item.
2. It appears with an `[AUCTION]` tag.
3. Buyers click it to open the bid selector.
4. Each bid beats the current one by the configured increment.
5. Outbid players are refunded.
6. A bid near the end extends the auction.
7. At the end the highest bidder gets the item, directly or to claim, and the seller is paid minus
   tax. With no bids the item goes back to the seller or is marked expired.

### Buy it now

```
/ah auction <starting_bid> <duration> <currency> <buy_it_now_price>
```

Taken with shift + right-click. The standing bidder is refunded after the sale is claimed under the
same lock as any other purchase.

```yaml
auction:
  buyout:
    enabled: false
```

Ships off. While it is off, auctions carry no buy-it-now price.

## Permissions

| Permission | Description |
|------------|-------------|
| `ah.admin` | Reload, purge, pah, serverlist, ban, unban |
| `ah.admin.remove` | Force-remove an item with shift + left-click |
| `ah.limit.<N>` | Fixed-price listings allowed |
| `ah.tax.<name>` | Tax rate for a rank, lowest wins |
| `ah.sell.<material>` | Sell a restricted material |

## Configuration

```yaml
# Extra names for /ah and /pah
command-aliases:
  ah:
    - auctionhouse
  pah: []

duration: 3d
maximum-price: 1000000000
default-currency: ""      # empty = primary Vault currency

tax:
  enabled: false
  rate: 0.05          # 5%
  type: SELLER        # SELLER or BUYER

auction:
  enabled: true
  default-duration: 1d
  min-duration: 1h
  max-duration: 7d
  min-starting-bid: 1
  min-bid-increment: 1.0
  min-bid-increment-percent: 0    # above 0 takes priority over the flat amount
  max-active-auctions: 5
  snipe-protection:
    enabled: true
    threshold: 5m
    extension: 5m
  # Button amounts by size of the current bid
  bid-increments:
    - threshold: 100
      values: [1, 10, 50]
    - threshold: 1000
      values: [10, 100, 500]
    - threshold: 10000
      values: [100, 1000, 5000]
    - threshold: -1          # fallback
      values: [1000, 10000, 50000]
  announce-bids: false
  announce-wins: true

blocked-materials: [ELYTRA]
blocked-names: [fuck]
blocked-anvil-names: [fuck]

announce-sale: true
announce-type: BOSSBAR    # CHAT, ACTION_BAR, TITLE, SUBTITLE, BOSSBAR

auto-return-expired: false

favorites:
  enabled: true

discord-sales:
  enabled: false
  webhook-url: ""
discord-admin:
  enabled: false
  webhook-url: ""

bans:
  default-reason: "Scam"
  exempt-admins: true            # ah.admin players cannot be banned
  cancel-listings-on-ban: true
  block-sell: true               # /ah sell and /ah auction
  block-buy: true
  block-bid: true
  block-open: false

server-listings:
  enabled: true
  display-name: "Server"
```

## Menus

In `plugins/XCore/addons/XAuctionHouse/guis/`:

| File | Screen |
|------|--------|
| `auction.yml` | Main browse (TimeSort, CategorySort, ListingTypeFilter, BackPage, MyItems, NextPage, Favorites, Search) |
| `my_items.yml` | A player's items (TimeSort, CategorySort, Filter, BackPage, BackToAH, NextPage) |
| `confirm.yml` | Purchase confirmation (Confirm, Cancel) |
| `search.yml` | Search results, plus ClearSearch |
| `player_auction.yml` | Admin view of a player |
| `viewer.yml` | Shulker viewer |
| `bid_amount.yml` | Bid selector (Plus1/10/100, Minus1/10/100, Display, Back, Confirm) |
| `bid_confirm.yml` | Bid confirmation |

Items accept `material`, `target-title`, `target-lore`, `target-button-on` and `off`, `permission`,
`sound`, `custom_model_data_value`, `item_model_key`, `actions`.

## PlaceholderAPI

| Placeholder | Returns |
|-------------|---------|
| `%xauctionhouse_sell_limit%` | Listings allowed |
| `%xauctionhouse_sell_count%` | Active listings |
| `%xauctionhouse_expired_count%` | Expired items |
| `%xauctionhouse_active_auctions%` | Running auctions |
| `%xauctionhouse_active_bids%` | Items where the player holds the highest bid |

## Languages

`lang/<code>.yml`, following XCore's `language` setting. English and French are bundled; an addon
without the chosen language falls back to English. New keys are appended at startup, existing values
are never overwritten.

## Web API

Base path `/api/xauctionhouse`.

| Endpoint | Returns |
|----------|---------|
| `GET /listings` | Active listings, with currency, type and bid info |
| `GET /sales` | Recent sales |
| `GET /stats` | Aggregate statistics |
| `GET /player/{name}` | A player's listings, sales and purchases |
| `GET /bans` | Auction-house bans |
| `POST /bans/add` | Ban (`{player, duration, reason}`, `def` = permanent) |
| `POST /bans/remove` | Unban (`{player}`) |

## Database

| Table | Contents |
|-------|----------|
| `xauctionhouse` | Listings |
| `xauctionhouse_bids` | Bid history |
| `xauctionhouse_favorites` | Favorites |
| `xauctionhouse_price_history` | Sale prices |
| `xauctionhouse_bans` | Auction-house bans |
