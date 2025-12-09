# Push Notifications

Get notified about print events via multiple channels including WhatsApp, Telegram, Discord, Email, and more.

## Supported Providers

| Provider | Description | Setup Complexity |
|----------|-------------|------------------|
| **WhatsApp** | Via CallMeBot service | Easy |
| **ntfy** | Self-hosted or ntfy.sh | Very Easy |
| **Pushover** | Dedicated push service | Easy |
| **Telegram** | Via Telegram Bot | Medium |
| **Email** | SMTP email | Medium |
| **Discord** | Channel webhooks | Easy |
| **Webhook** | Generic HTTP POST | Flexible |

---

## Adding a Notification Provider

1. Go to **Settings** > **Notifications**
2. Click **Add Provider**
3. Select provider type
4. Enter required configuration
5. Click **Send Test** to verify
6. Configure event triggers
7. Click **Add**

---

## Provider Setup Guides

### WhatsApp (CallMeBot)

Free WhatsApp notifications via CallMeBot:

1. Add CallMeBot to contacts: **+34 644 51 95 23**
2. Send via WhatsApp: `I allow callmebot to send me messages`
3. You'll receive an API key
4. In Bambuddy, enter:
   - **Phone Number**: Your number with country code (e.g., +1234567890)
   - **API Key**: The key from CallMeBot

### ntfy

Simple topic-based notifications:

1. Choose a unique topic name (e.g., `my-printer-xyz123`)
2. Subscribe on your phone:
   - Install ntfy app ([Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) / [iOS](https://apps.apple.com/app/ntfy/id1625396347))
   - Subscribe to your topic
3. In Bambuddy, enter:
   - **Topic**: Your chosen topic name
   - **Server**: `https://ntfy.sh` (default) or your self-hosted server

> No account needed for ntfy.sh public server!

### Pushover

Professional push notification service:

1. Create account at [pushover.net](https://pushover.net/)
2. Install Pushover app on your device
3. Create an Application in Pushover dashboard
4. In Bambuddy, enter:
   - **User Key**: From your Pushover account
   - **API Token**: From your Application

### Telegram

Via Telegram Bot:

1. Message [@BotFather](https://t.me/BotFather) on Telegram
2. Send `/newbot` and follow prompts
3. Save the **Bot Token** provided
4. Message [@userinfobot](https://t.me/userinfobot) to get your **Chat ID**
5. In Bambuddy, enter:
   - **Bot Token**: From BotFather
   - **Chat ID**: Your user or group chat ID

> For group notifications, add the bot to the group and use the group's chat ID.

### Email (SMTP)

Send notifications via email:

1. Gather your SMTP server details
2. In Bambuddy, enter:
   - **SMTP Server**: e.g., `smtp.gmail.com`
   - **Port**: 587 (STARTTLS), 465 (SSL), or 25 (None)
   - **Security**: STARTTLS, SSL, or None
   - **Username**: Your email address
   - **Password**: Your password or app password
   - **From Address**: Sender email
   - **To Address**: Recipient email

#### Gmail Setup
1. Enable 2-Factor Authentication on your Google account
2. Generate an [App Password](https://myaccount.google.com/apppasswords)
3. Use the app password (not your regular password)
4. Server: `smtp.gmail.com`, Port: 587, Security: STARTTLS

### Discord

Via channel webhooks:

1. In Discord, go to channel settings
2. Navigate to **Integrations** > **Webhooks**
3. Click **New Webhook**
4. Customize name/avatar if desired
5. Click **Copy Webhook URL**
6. In Bambuddy, paste the **Webhook URL**

### Webhook (Generic)

For custom integrations (Home Assistant, IFTTT, etc.):

1. Enter your webhook URL
2. Optionally add custom headers (e.g., Authorization)
3. Bambuddy sends JSON payloads:

```json
{
  "event": "print_complete",
  "printer": "Workshop X1C",
  "filename": "benchy.3mf",
  "duration": "2h 15m",
  "timestamp": "2024-01-15T14:30:00Z"
}
```

---

## Event Triggers

Configure which events send notifications:

| Event | Description |
|-------|-------------|
| **Print Started** | When a print job begins |
| **Print Completed** | When a print finishes successfully |
| **Print Failed** | When a print fails or errors out |
| **Print Stopped** | When you manually cancel a print |
| **Progress Milestones** | At 25%, 50%, and 75% progress |
| **Printer Offline** | When a printer disconnects |
| **Printer Error** | When HMS errors are detected |
| **Low Filament** | When filament is running low |

Enable/disable each event per provider.

---

## Quiet Hours

Suppress notifications during sleep or focus time:

1. Enable **Quiet Hours** toggle
2. Set **Start Time** (e.g., 22:00)
3. Set **End Time** (e.g., 07:00)

Notifications during quiet hours are silently skipped.

---

## Per-Printer Filtering

Limit notifications to specific printers:

1. Open provider settings
2. Find **Printer** dropdown
3. Select a specific printer (or "All Printers")

Only events from the selected printer(s) trigger notifications.

---

## Daily Digest

Batch notifications into a daily summary:

1. Enable **Daily Digest** toggle
2. Set **Digest Time** (e.g., 08:00)

### How It Works
- Individual events are collected (not sent immediately)
- At digest time, one summary notification is sent
- Summary includes counts and details of all events

### Example Digest
```
Daily Print Summary:
- 3 prints completed
- 1 print failed
- Total print time: 8h 45m
- Filament used: 245g
```

---

## Message Templates

Customize notification messages using variables:

### Accessing Templates
1. Go to **Settings** > **Notifications**
2. Click **Templates** tab
3. Click any event type to edit

### Using Variables
Insert dynamic content using `{variable}` syntax:
- Click variable buttons to insert
- Preview with sample data
- Save to apply changes

### Available Variables

**Print Start**
- `{printer}` - Printer name
- `{filename}` - Print filename
- `{estimated_time}` - Estimated duration

**Print Complete**
- `{printer}`, `{filename}`
- `{duration}` - Actual print time
- `{filament_grams}` - Filament used

**Print Failed**
- `{printer}`, `{filename}`, `{duration}`
- `{reason}` - Failure reason

**Print Progress**
- `{printer}`, `{filename}`
- `{progress}` - Percentage complete
- `{remaining_time}` - Time remaining

**Printer Offline**
- `{printer}`

**Printer Error**
- `{printer}`
- `{error_type}` - HMS error type
- `{error_detail}` - Error description

**Filament Low**
- `{printer}`
- `{slot}` - AMS slot number
- `{remaining_percent}` - Filament remaining
- `{color}` - Filament color

**Common Variables** (all events)
- `{timestamp}` - Event time
- `{app_name}` - "Bambuddy"

### Reset to Default
Click the reset button on any template to restore the original message.

---

## Testing Notifications

Before relying on notifications:

1. Configure your provider
2. Click **Send Test**
3. Verify you receive the test message
4. If it fails, check configuration and try again

---

## Tips

- Use **ntfy** for the simplest setup (no account needed)
- Enable **quiet hours** to avoid middle-of-night alerts
- Set up **multiple providers** for redundancy
- Use **progress milestones** for long prints
- Configure **printer filtering** if you have multiple printers
- Customize **templates** to include only the info you need
