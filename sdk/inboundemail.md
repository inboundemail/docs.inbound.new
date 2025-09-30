---
title: "Inbound Email SDK"
description: "Complete SDK reference for the Inbound Email API with Resend-compatible interface"
---

# Inbound Email SDK

The official TypeScript/JavaScript SDK for Inbound Email. Built with a Resend-compatible API for seamless migration and familiar developer experience.

## Installation

<CodeGroup>
```bash bun
bun add inboundemail
```
</CodeGroup>

## Quick Start

Initialize the SDK with your API key:

```typescript
import { Inbound } from 'inboundemail'

const inbound = new Inbound('your_api_key_here')
```

<Tip>
Get your API key from your [Inbound Email dashboard](https://inbound.new/dashboard/settings/api-keys)
</Tip>

## Core Resources

### Emails

Send, retrieve, and manage emails with full MIME support.

#### Send Email

Send a transactional email with support for HTML, text, attachments, and custom headers.

```typescript
// POST /api/v2/emails
const { data, error } = await inbound.emails.send({
  from: 'hello@yourdomain.com',
  to: 'recipient@example.com',
  subject: 'Welcome to Inbound',
  html: '<p>Thanks for signing up!</p>',
  text: 'Thanks for signing up!',
})

console.log(data?.id) // Email ID
console.log(data?.messageId) // AWS SES Message ID
```

<ParamField body="from" type="string" required>
Sender email address. Supports both formats: `email@domain.com` or `Name <email@domain.com>`
</ParamField>

<ParamField body="to" type="string | string[]" required>
Recipient email address(es)
</ParamField>

<ParamField body="subject" type="string" required>
Email subject line
</ParamField>

<ParamField body="html" type="string">
HTML version of the email content
</ParamField>

<ParamField body="text" type="string">
Plain text version of the email content
</ParamField>

<ParamField body="cc" type="string | string[]">
Carbon copy recipients
</ParamField>

<ParamField body="bcc" type="string | string[]">
Blind carbon copy recipients
</ParamField>

<ParamField body="replyTo" type="string | string[]">
Reply-To email address(es)
</ParamField>

<ParamField body="headers" type="Record<string, string>">
Custom email headers
</ParamField>

<ParamField body="attachments" type="Attachment[]">
File attachments (see [Attachments](#attachments) section)
</ParamField>

<ParamField body="tags" type="Array<{name: string, value: string}>">
Email tags for tracking and filtering
</ParamField>

<ResponseField name="id" type="string">
Unique identifier for the sent email
</ResponseField>

<ResponseField name="messageId" type="string">
AWS SES Message ID for tracking
</ResponseField>

#### Get Email

Retrieve a sent email by ID.

```typescript
// GET /api/v2/emails/{id}
const { data, error } = await inbound.emails.get('email_id')

console.log(data?.subject)
console.log(data?.from)
console.log(data?.to)
console.log(data?.last_event) // 'pending' | 'delivered' | 'failed'
```

<ParamField path="id" type="string" required>
The email ID to retrieve
</ParamField>

<ResponseField name="object" type="string">
Always returns `"email"`
</ResponseField>

<ResponseField name="id" type="string">
Email identifier
</ResponseField>

<ResponseField name="from" type="string">
Sender email address
</ResponseField>

<ResponseField name="to" type="string[]">
Recipient email addresses
</ResponseField>

<ResponseField name="subject" type="string">
Email subject
</ResponseField>

<ResponseField name="html" type="string | null">
HTML content
</ResponseField>

<ResponseField name="text" type="string | null">
Plain text content
</ResponseField>

<ResponseField name="cc" type="(string | null)[]">
CC recipients
</ResponseField>

<ResponseField name="bcc" type="(string | null)[]">
BCC recipients
</ResponseField>

<ResponseField name="reply_to" type="(string | null)[]">
Reply-To addresses
</ResponseField>

<ResponseField name="created_at" type="string">
ISO 8601 timestamp
</ResponseField>

<ResponseField name="last_event" type="string">
Last status event: `pending`, `delivered`, or `failed`
</ResponseField>

#### Reply to Email

Reply to an inbound email or thread with automatic threading support.

```typescript
// POST /api/v2/emails/{id}/reply-new
const { data, error } = await inbound.emails.reply('email_id', {
  from: 'support@yourdomain.com',
  text: 'Thanks for reaching out!',
  html: '<p>Thanks for reaching out!</p>',
})

console.log(data?.id) // Reply email ID
console.log(data?.messageId) // Inbound message ID (for threading)
console.log(data?.awsMessageId) // AWS SES Message ID
console.log(data?.repliedToEmailId) // Original email ID
console.log(data?.isThreadReply) // Whether this was a thread reply
```

<ParamField path="id" type="string" required>
Email ID or thread ID to reply to. When given a thread ID, replies to the latest message.
</ParamField>

<ParamField body="from" type="string" required>
Your reply-from address (must be from a verified domain). Supports both formats: `email@domain.com` or `Name <email@domain.com>`
</ParamField>

<ParamField body="to" type="string | string[]">
Optional - defaults to original sender
</ParamField>

<ParamField body="subject" type="string">
Optional - defaults to "Re: [original subject]"
<Note>
You probably don't want to cusomize this as this might break Gmails threading logic.
</Note>
</ParamField>

<ParamField body="text" type="string">
Plain text reply content
</ParamField>

<ParamField body="html" type="string">
HTML reply content
</ParamField>

<ParamField body="replyAll" type="boolean" default="false">
Reply to all original recipients including CC addresses
</ParamField>

<ParamField body="headers" type="Record<string, string>">
Custom email headers
</ParamField>

<ParamField body="attachments" type="Attachment[]">
File attachments for the reply
</ParamField>

<ParamField body="tags" type="Array<{name: string, value: string}>">
Email tags for tracking
</ParamField>

<ResponseField name="id" type="string">
Reply email ID
</ResponseField>

<ResponseField name="messageId" type="string">
Inbound message ID used for threading
</ResponseField>

<ResponseField name="awsMessageId" type="string">
AWS SES Message ID
</ResponseField>

<ResponseField name="repliedToEmailId" type="string">
The actual email ID that was replied to
</ResponseField>

<ResponseField name="repliedToThreadId" type="string">
The thread ID (if replying to a thread)
</ResponseField>

<ResponseField name="isThreadReply" type="boolean">
Whether this was a reply to a thread ID vs direct email ID
</ResponseField>

<Warning>
Reply functionality requires the email to be in your mailbox. You can only reply to emails you've received. The `from` address cannot be `agent@inbnd.dev` for replies.
</Warning>

#### Schedule Email

Schedule an email to be sent at a future time.

```typescript
// POST /api/v2/emails/schedule
const { data, error } = await inbound.emails.schedule({
  from: 'hello@yourdomain.com',
  to: 'recipient@example.com',
  subject: 'Scheduled Newsletter',
  html: '<p>Your weekly update</p>',
  scheduled_at: '2024-12-31T09:00:00Z',
  timezone: 'America/New_York'
})

console.log(data?.id) // Scheduled email ID
console.log(data?.scheduled_at) // Normalized timestamp
console.log(data?.status) // 'scheduled'
```

<ParamField body="scheduled_at" type="string" required>
ISO 8601 timestamp or natural language ("in 1 hour", "tomorrow at 9am")
</ParamField>

<ParamField body="timezone" type="string" default="UTC">
Timezone for natural language parsing
</ParamField>

<Info>
All other parameters match the `send` method
</Info>

### Threads

Manage email conversation threads with intelligent grouping.

#### List Threads

Get all conversation threads in your mailbox.

```typescript
// GET /api/v2/threads
const { data, error } = await inbound.threads.list({
  page: 1,
  limit: 50,
  unreadOnly: true
})

console.log(data?.threads.length)
console.log(data?.pagination.total)
```

<ParamField query="page" type="number" default="1">
Page number (1-indexed)
</ParamField>

<ParamField query="limit" type="number" default="50">
Threads per page (max: 100)
</ParamField>

<ParamField query="search" type="string">
Search thread subjects and content
</ParamField>

<ParamField query="unreadOnly" type="boolean" default="false">
Show only threads with unread messages
</ParamField>

<ParamField query="archivedOnly" type="boolean" default="false">
Show only archived threads
</ParamField>

#### Get Thread

Retrieve all messages in a conversation thread.

```typescript
// GET /api/v2/threads/{id}
const { data, error } = await inbound.threads.get('thread_id')

console.log(data?.thread.messageCount)
console.log(data?.messages.length)
console.log(data?.thread.participantEmails)
```

<ResponseField name="thread" type="object">
Thread metadata including participant emails, message count, and timestamps
</ResponseField>

<ResponseField name="messages" type="ThreadMessage[]">
All messages in the thread, sorted chronologically
</ResponseField>

<ResponseField name="totalCount" type="number">
Total message count in thread
</ResponseField>

#### Thread Actions

Perform actions on entire threads.

```typescript
// POST /api/v2/threads/{id}/actions
const { data, error } = await inbound.threads.action('thread_id', {
  action: 'mark_as_read'
})

// Available actions:
// - 'mark_as_read'
// - 'mark_as_unread'
// - 'archive'
// - 'unarchive'
```

### Domains

Manage email domains for sending and receiving.

#### Create Domain

Add a new domain for email handling.

```typescript
// POST /api/v2/domains
const { data, error } = await inbound.domains.create({
  domain: 'yourdomain.com'
})

console.log(data?.id)
console.log(data?.status) // 'pending' | 'verified' | 'failed'
console.log(data?.dnsRecords) // DNS records to add
```

<ParamField body="domain" type="string" required>
Domain name to add (e.g., `example.com`)
</ParamField>

<ResponseField name="id" type="string">
Unique domain identifier
</ResponseField>

<ResponseField name="domain" type="string">
The domain name
</ResponseField>

<ResponseField name="status" type="string">
Verification status: `pending`, `verified`, or `failed`
</ResponseField>

<ResponseField name="dnsRecords" type="DnsRecord[]">
DNS records to add to your DNS provider

<Expandable title="DNS Record Properties">
  <ResponseField name="type" type="string">
  Record type: `TXT`, `MX`, or `CNAME`
  </ResponseField>
  
  <ResponseField name="name" type="string">
  Record name/host
  </ResponseField>
  
  <ResponseField name="value" type="string">
  Record value/content
  </ResponseField>
  
  <ResponseField name="isRequired" type="boolean">
  Whether this record is required for verification
  </ResponseField>
</Expandable>
</ResponseField>

#### List Domains

Get all domains with optional filtering.

```typescript
// GET /api/v2/domains
const { data, error } = await inbound.domains.list({
  limit: 50,
  status: 'verified',
  check: true // Recheck Statuses on domains and DNS records (this will take a few seconds)
})

console.log(data?.data.length)
console.log(data?.meta.verifiedCount)
```

<ParamField query="limit" type="number" default="50">
Number of domains to return (max: 100)
</ParamField>

<ParamField query="offset" type="number" default="0">
Pagination offset
</ParamField>

<ParamField query="status" type="string">
Filter by status: `pending`, `verified`, or `failed`
</ParamField>

<ParamField query="canReceive" type="boolean">
Filter by email receiving capability
</ParamField>

<ParamField query="check" type="boolean" default="false">
Recheck Statuses on domains and DNS records (this will take a few seconds)
</ParamField>

#### Get Domain

Retrieve detailed domain information.

```typescript
// GET /api/v2/domains/{id}
const { data, error } = await inbound.domains.get('domain_id', {
  check: true // Recheck Statuses on domains and DNS records (this will take a few seconds)
})

console.log(data?.status)
console.log(data?.stats.totalEmailAddresses)
console.log(data?.verificationCheck?.isFullyVerified)
```

<ParamField path="id" type="string" required>
Domain ID
</ParamField>

<ParamField query="check" type="boolean" default="false">
Recheck Statuses on domains and DNS records (this will take a few seconds)
</ParamField>

#### Update Domain

Configure domain catch-all settings.

```typescript
// PUT /api/v2/domains/{id}
const { data, error } = await inbound.domains.update('domain_id', {
  isCatchAllEnabled: true,
  catchAllEndpointId: 'endpoint_id'
})
```

#### Delete Domain

Remove a domain and all associated resources.

```typescript
// DELETE /api/v2/domains/{id}
const { data, error } = await inbound.domains.delete('domain_id')

console.log(data?.deletedResources.emailAddresses)
console.log(data?.deletedResources.dnsRecords)
```

<Warning>
Deleting a domain removes all email addresses, DNS records, and blocks associated with it. This action cannot be undone.
</Warning>

#### Initialize Domain Authentication

Set up domain authentication (DKIM, SPF, DMARC).

```typescript
// POST /api/v2/domains/{id}/auth
const { data, error } = await inbound.domains.initAuth('domain_id', {
  mailFromDomain: 'mail',
  generateSpf: true,
  generateDmarc: true
})

console.log(data?.records) // DNS records to add
console.log(data?.dkimTokens) // DKIM verification tokens
```

#### Verify Domain Authentication

Check domain authentication status.

```typescript
// PATCH /api/v2/domains/{id}/auth
const { data, error } = await inbound.domains.verifyAuth('domain_id')

console.log(data?.overallStatus) // 'verified' | 'pending' | 'failed'
console.log(data?.sesStatus.identityVerified)
console.log(data?.sesStatus.dkimVerified)
console.log(data?.summary.verifiedRecords)
```

#### Get DNS Records

Retrieve configured DNS records for a domain.

```typescript
// GET /api/v2/domains/{id}/dns-records
const { data, error } = await inbound.domains.getDnsRecords('domain_id')

console.log(data?.records.length)
console.log(data?.records.filter(r => r.isVerified))
```

### Email Addresses

Manage individual email addresses for receiving mail.

#### Create Email Address

Add a new email address to receive mail.

```typescript
// POST /api/v2/email-addresses
const { data, error } = await inbound.emailAddresses.create({
  address: 'hello@yourdomain.com',
  domainId: 'domain_id',
  endpointId: 'endpoint_id',
  isActive: true
})

console.log(data?.id)
console.log(data?.routing.type) // 'endpoint' | 'webhook' | 'none'
```

<ParamField body="address" type="string" required>
Full email address
</ParamField>

<ParamField body="domainId" type="string" required>
Domain ID (must belong to your account)
</ParamField>

<ParamField body="endpointId" type="string">
Endpoint ID to route emails to
</ParamField>

<ParamField body="webhookId" type="string">
Webhook ID to route emails to (legacy - use endpointId instead)
</ParamField>

<ParamField body="isActive" type="boolean" default="true">
Whether the email address is active
</ParamField>

#### List Email Addresses

Get all email addresses with filtering.

```typescript
// GET /api/v2/email-addresses
const { data, error } = await inbound.emailAddresses.list({
  limit: 50,
  domainId: 'domain_id',
  isActive: true
})

console.log(data?.data.length)
console.log(data?.pagination.total)
```

#### Get Email Address

Retrieve email address details.

```typescript
// GET /api/v2/email-addresses/{id}
const { data, error } = await inbound.emailAddresses.get('address_id')

console.log(data?.address)
console.log(data?.domain.name)
console.log(data?.routing)
```

#### Update Email Address

Modify email address settings.

```typescript
// PUT /api/v2/email-addresses/{id}
const { data, error } = await inbound.emailAddresses.update('address_id', {
  endpointId: 'new_endpoint_id',
  isActive: true
})
```

#### Delete Email Address

Remove an email address.

```typescript
// DELETE /api/v2/email-addresses/{id}
const { data, error } = await inbound.emailAddresses.delete('address_id')

console.log(data?.cleanup.sesRuleUpdated)
```

### Endpoints

Configure endpoints for email delivery (webhooks, email forwarding, email groups).

#### Create Endpoint

Create a new endpoint for receiving emails.

<Tabs>
<Tab title="Webhook">
```typescript
// POST /api/v2/endpoints
const { data, error } = await inbound.endpoints.create({
  name: 'Production Webhook',
  type: 'webhook',
  description: 'Main webhook for processing emails',
  config: {
    url: 'https://yourdomain.com/webhook',
    timeout: 30,
    retryAttempts: 3,
    headers: {
      'X-Custom-Header': 'value'
    }
  }
})
```
</Tab>

<Tab title="Email Forward">
```typescript
const { data, error } = await inbound.endpoints.create({
  name: 'Personal Email Forward',
  type: 'email',
  description: 'Forward to personal inbox',
  config: {
    forwardTo: 'personal@gmail.com',
    preserveHeaders: true,
    subjectPrefix: '[Forwarded]'
  }
})
```
</Tab>

<Tab title="Email Group">
```typescript
const { data, error } = await inbound.endpoints.create({
  name: 'Team Distribution',
  type: 'email_group',
  description: 'Forward to entire team',
  config: {
    emails: [
      'alice@team.com',
      'bob@team.com',
      'charlie@team.com'
    ]
  }
})
```
</Tab>
</Tabs>

<ParamField body="name" type="string" required>
Endpoint display name
</ParamField>

<ParamField body="type" type="string" required>
Endpoint type: `webhook`, `email`, or `email_group`
</ParamField>

<ParamField body="config" type="object" required>
Type-specific configuration object
</ParamField>

<ParamField body="description" type="string">
Optional description
</ParamField>

#### List Endpoints

Get all endpoints with delivery statistics.

```typescript
// GET /api/v2/endpoints
const { data, error } = await inbound.endpoints.list({
  limit: 50,
  type: 'webhook',
  active: true,
  sortBy: 'newest'
})

console.log(data?.data.length)
for (const endpoint of data?.data || []) {
  console.log(endpoint.deliveryStats.total)
  console.log(endpoint.deliveryStats.successful)
}
```

#### Get Endpoint

Retrieve endpoint details with delivery history.

```typescript
// GET /api/v2/endpoints/{id}
const { data, error } = await inbound.endpoints.get('endpoint_id')

console.log(data?.deliveryStats)
console.log(data?.recentDeliveries)
console.log(data?.associatedEmails)
console.log(data?.catchAllDomains)
```

#### Update Endpoint

Modify endpoint configuration.

```typescript
// PUT /api/v2/endpoints/{id}
const { data, error } = await inbound.endpoints.update('endpoint_id', {
  name: 'Updated Webhook',
  config: {
    url: 'https://new-url.com/webhook',
    timeout: 60
  },
  isActive: false
})
```

#### Delete Endpoint

Remove an endpoint and update routing.

```typescript
// DELETE /api/v2/endpoints/{id}
const { data, error } = await inbound.endpoints.delete('endpoint_id')

console.log(data?.cleanup.emailAddressesUpdated)
console.log(data?.cleanup.deliveriesDeleted)
```

#### Test Endpoint

Send a test payload to verify endpoint configuration.

```typescript
// POST /api/v2/endpoints/{id}/test
const { data, error } = await inbound.endpoints.test('endpoint_id', {
  webhookFormat: 'inbound',
  overrideUrl: 'https://temp-test-url.com'
})

console.log(data?.success)
console.log(data?.responseTime)
console.log(data?.statusCode)
```

<ParamField body="webhookFormat" type="string" default="inbound">
Webhook payload format: `inbound`, `discord`, or `slack`
</ParamField>

<ParamField body="overrideUrl" type="string">
Test a different URL without updating the endpoint
</ParamField>

### Attachments

Work with email attachments efficiently.

#### Send Email with Attachments

<CodeGroup>
```typescript Base64
const { data, error } = await inbound.emails.send({
  from: 'hello@yourdomain.com',
  to: 'recipient@example.com',
  subject: 'Invoice for January',
  html: '<p>Please find your invoice attached.</p>',
  attachments: [
    {
      content: base64String,
      filename: 'invoice.pdf',
      contentType: 'application/pdf'
    }
  ]
})
```

```typescript Remote File
const { data, error } = await inbound.emails.send({
  from: 'hello@yourdomain.com',
  to: 'recipient@example.com',
  subject: 'Monthly Report',
  html: '<p>Report attached</p>',
  attachments: [
    {
      path: 'https://cdn.example.com/reports/monthly.pdf',
      filename: 'monthly-report.pdf'
    }
  ]
})
```

```typescript CID Embedding
const { data, error } = await inbound.emails.send({
  from: 'hello@yourdomain.com',
  to: 'recipient@example.com',
  subject: 'Welcome Email',
  html: '<p>Welcome! <img src="cid:logo" alt="Logo" /></p>',
  attachments: [
    {
      content: logoBase64,
      filename: 'logo.png',
      contentType: 'image/png',
      content_id: 'logo' // Embed inline using CID
    }
  ]
})
```
</CodeGroup>

<ParamField body="attachments[].content" type="string">
Base64 encoded file content (mutually exclusive with `path`)
</ParamField>

<ParamField body="attachments[].path" type="string">
Remote file URL to fetch (mutually exclusive with `content`)
</ParamField>

<ParamField body="attachments[].filename" type="string" required>
Display filename for the attachment
</ParamField>

<ParamField body="attachments[].contentType" type="string">
MIME type (auto-detected if not provided)
</ParamField>

<ParamField body="attachments[].content_id" type="string">
Content ID for embedding images inline using `cid:` references
</ParamField>

<Check>
**Attachment Limits**
- Maximum 25MB per attachment
- Maximum 40MB total email size
- Maximum 20 attachments per email
- Supports common document, image, and archive formats
</Check>

#### Download Attachment

Download an attachment from a received email.

```typescript
// GET /api/v2/attachments/{id}/{filename}
const attachmentUrl = `https://inbound.new/api/v2/attachments/${emailId}/${filename}`

// Or use the SDK helper
const response = await fetch(attachmentUrl, {
  headers: {
    'Authorization': `Bearer ${apiKey}`
  }
})

const blob = await response.blob()
```

<ParamField path="emailId" type="string" required>
Structured email ID containing the attachment
</ParamField>

<ParamField path="filename" type="string" required>
Attachment filename (URL encoded)
</ParamField>

## Advanced Features

### Idempotency

Prevent duplicate sends using idempotency keys.

```typescript
const { data, error } = await inbound.emails.send(
  {
    from: 'hello@yourdomain.com',
    to: 'recipient@example.com',
    subject: 'Important Email',
    text: 'Critical message'
  },
  {
    headers: {
      'Idempotency-Key': 'unique-operation-id'
    }
  }
)

// Second call with same key returns the same email ID
```

<Info>
Idempotency keys are stored for 24 hours and prevent accidental duplicate sends
</Info>

### Email Threading

Proper email threading with RFC 5322 compliance.

```typescript
// Original email
const { data: original } = await inbound.emails.send({
  from: 'support@yourdomain.com',
  to: 'customer@example.com',
  subject: 'Ticket #123',
  text: 'How can we help?'
})

// Reply maintains thread
const { data: reply } = await inbound.emails.reply(original.id, {
  from: 'support@yourdomain.com',
  text: 'Following up on your request',
  includeOriginal: true // Quotes original message
})

// Thread is automatically maintained via Message-ID headers
```

### Custom Headers

Add custom headers for tracking and routing.

```typescript
const { data, error } = await inbound.emails.send({
  from: 'hello@yourdomain.com',
  to: 'recipient@example.com',
  subject: 'Custom Headers Example',
  text: 'Email with custom headers',
  headers: {
    'X-Campaign-ID': 'summer-2024',
    'X-Priority': 'High',
    'X-Custom-Tracking': 'abc123'
  }
})
```

### Email Scheduling

Schedule emails with flexible time specifications.

<CodeGroup>
```typescript ISO 8601
const { data, error } = await inbound.emails.schedule({
  from: 'newsletter@yourdomain.com',
  to: 'subscribers@example.com',
  subject: 'Weekly Newsletter',
  html: '<h1>This Week</h1>',
  scheduled_at: '2024-12-25T09:00:00Z',
  timezone: 'UTC'
})
```

```typescript Natural Language
const { data, error } = await inbound.emails.schedule({
  from: 'reminders@yourdomain.com',
  to: 'user@example.com',
  subject: 'Meeting Reminder',
  text: 'Your meeting is tomorrow',
  scheduled_at: 'tomorrow at 9am',
  timezone: 'America/New_York'
})
```
</CodeGroup>

### Webhook Formats

Support for multiple webhook payload formats.

<Tabs>
<Tab title="Inbound (Default)">
```typescript
{
  event: 'email.received',
  timestamp: '2024-01-15T10:30:00Z',
  email: {
    id: 'email_123',
    messageId: '<abc@mail.example.com>',
    from: {
      text: 'John Doe <john@example.com>',
      addresses: [{ name: 'John Doe', address: 'john@example.com' }]
    },
    to: {
      text: 'support@yourdomain.com',
      addresses: [{ address: 'support@yourdomain.com' }]
    },
    subject: 'Help Request',
    receivedAt: '2024-01-15T10:30:00Z',
    parsedData: {
      textBody: 'I need help with...',
      htmlBody: '<p>I need help with...</p>',
      attachments: [],
      headers: {}
    },
    cleanedContent: {
      html: '<p>I need help with...</p>',
      text: 'I need help with...',
      hasHtml: true,
      hasText: true
    }
  },
  endpoint: {
    id: 'endpoint_123',
    name: 'Production Webhook',
    type: 'webhook'
  }
}
```
</Tab>

<Tab title="Discord">
```typescript
{
  content: 'New email from john@example.com',
  embeds: [
    {
      title: 'Help Request',
      description: 'I need help with...',
      fields: [
        { name: 'From', value: 'john@example.com' },
        { name: 'To', value: 'support@yourdomain.com' },
        { name: 'Received', value: '2024-01-15T10:30:00Z' }
      ],
      color: 5814783
    }
  ]
}
```
</Tab>

<Tab title="Slack">
```typescript
{
  text: 'New email from john@example.com',
  blocks: [
    {
      type: 'header',
      text: { type: 'plain_text', text: 'Help Request' }
    },
    {
      type: 'section',
      text: { type: 'mrkdwn', text: 'I need help with...' }
    },
    {
      type: 'context',
      elements: [
        { type: 'mrkdwn', text: '*From:* john@example.com' },
        { type: 'mrkdwn', text: '*To:* support@yourdomain.com' }
      ]
    }
  ]
}
```
</Tab>
</Tabs>

## Error Handling

The SDK uses a consistent error response format.

```typescript
const { data, error } = await inbound.emails.send({
  from: 'hello@yourdomain.com',
  to: 'invalid-email',
  subject: 'Test',
  text: 'Test'
})

if (error) {
  console.error('Error code:', error.code)
  console.error('Error message:', error.message)
  console.error('Details:', error.details)
}
```

### Common Error Codes

| Code | Description | Resolution |
|------|-------------|------------|
| `401` | Unauthorized | Check API key is valid |
| `403` | Forbidden | Verify domain ownership |
| `404` | Not Found | Check resource ID exists |
| `400` | Bad Request | Validate request parameters |
| `429` | Rate Limited | Reduce request rate or upgrade plan |
| `500` | Server Error | Contact support if persists |

## Rate Limits

Inbound Email enforces rate limits to ensure service quality.

<CardGroup cols={2}>
<Card title="Free Tier" icon="gauge-simple-low">
- 100 emails/day sent
- 1,000 emails/month received
- 3 domains
- 10 email addresses
</Card>

<Card title="Pro Tier" icon="gauge-simple-high">
- 10,000 emails/day sent
- Unlimited received
- 10 domains
- 100 email addresses
</Card>
</CardGroup>

<Warning>
Rate limit headers are included in API responses:
- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`
</Warning>

## TypeScript Support

Full TypeScript support with comprehensive type definitions.

```typescript
import { 
  Inbound,
  type SendEmailRequest,
  type SendEmailResponse,
  type InboundWebhookPayload
} from 'inboundemail'

const inbound = new Inbound(process.env.INBOUND_API_KEY!)

// Type-safe email sending
const request: SendEmailRequest = {
  from: 'hello@yourdomain.com',
  to: ['recipient@example.com'],
  subject: 'Typed Email',
  text: 'Fully typed SDK!'
}

const response: SendEmailResponse = await inbound.emails.send(request)
```

## Migration from Resend

Switching from Resend is seamless due to API compatibility.

<Steps>
<Step title="Install Inbound SDK">
```bash
npm install inboundemail
```
</Step>

<Step title="Update Imports">
```typescript
// Before
import { Resend } from 'resend'
const resend = new Resend(process.env.RESEND_API_KEY)

// After
import { Inbound } from 'inboundemail'
const inbound = new Inbound(process.env.INBOUND_API_KEY)
```
</Step>

<Step title="Update Method Calls">
Most Resend methods work identically:

```typescript
// Resend syntax works with Inbound
await inbound.emails.send({ ... })
await inbound.domains.create({ ... })
```
</Step>

<Step title="Add Inbound-Specific Features">
Take advantage of additional features:

```typescript
// Reply to inbound emails
await inbound.emails.reply(emailId, { ... })

// Manage threads
await inbound.threads.list({ ... })

// Configure endpoints
await inbound.endpoints.create({ ... })
```
</Step>
</Steps>

<Check>
Inbound maintains backward compatibility with Resend's API while adding powerful inbound email capabilities
</Check>

## Examples

### Send Transactional Email

```typescript
const { data, error } = await inbound.emails.send({
  from: 'noreply@yourdomain.com',
  to: 'user@example.com',
  subject: 'Welcome to Our Service!',
  html: `
    <div>
      <h1>Welcome!</h1>
      <p>Thanks for signing up. Get started by verifying your email.</p>
      <a href="https://yourdomain.com/verify?token=abc123">Verify Email</a>
    </div>
  `,
  text: 'Welcome! Thanks for signing up. Verify your email at: https://yourdomain.com/verify?token=abc123',
  tags: [
    { name: 'category', value: 'welcome' },
    { name: 'user_id', value: '12345' }
  ]
})

if (error) {
  console.error('Failed to send:', error)
} else {
  console.log('Email sent:', data?.id)
}
```

### Handle Inbound Email

```typescript
// Set up webhook endpoint
const { data: endpoint } = await inbound.endpoints.create({
  name: 'Support Webhook',
  type: 'webhook',
  config: {
    url: 'https://yourdomain.com/webhooks/email',
    timeout: 30
  }
})

// Create email address
const { data: emailAddress } = await inbound.emailAddresses.create({
  address: 'support@yourdomain.com',
  domainId: 'domain_id',
  endpointId: endpoint.id
})

// Your webhook handler receives:
app.post('/webhooks/email', async (req, res) => {
  const payload = req.body
  
  console.log('New email from:', payload.email.from.text)
  console.log('Subject:', payload.email.subject)
  console.log('Body:', payload.email.cleanedContent.text)
  
  // Reply automatically
  await inbound.emails.reply(payload.email.id, {
    from: 'support@yourdomain.com',
    text: 'Thanks for contacting us! We\'ll respond within 24 hours.'
  })
  
  res.json({ success: true })
})
```

### Process Email Thread

```typescript
// Get conversation thread
const { data: thread } = await inbound.threads.get('thread_id')

console.log('Thread participants:', thread?.thread.participantEmails)
console.log('Total messages:', thread?.thread.messageCount)

// Display all messages
for (const message of thread?.messages || []) {
  console.log(`[${message.type}] ${message.from}: ${message.subject}`)
  console.log(`  Date: ${message.date}`)
  console.log(`  Preview: ${message.content.textBody?.substring(0, 100)}`)
}

// Mark entire thread as read
await inbound.threads.action(thread?.thread.id, {
  action: 'mark_as_read'
})
```

### Verify Domain

```typescript
// Step 1: Create domain
const { data: domain } = await inbound.domains.create({
  domain: 'yourdomain.com'
})

console.log('Add these DNS records:')
for (const record of domain?.dnsRecords || []) {
  console.log(`${record.type} ${record.name} = ${record.value}`)
}

// Step 2: Initialize authentication
const { data: auth } = await inbound.domains.initAuth(domain.id, {
  mailFromDomain: 'mail',
  generateSpf: true,
  generateDmarc: true
})

console.log('DKIM tokens:', auth?.dkimTokens)

// Step 3: Verify after adding DNS records
const { data: verification } = await inbound.domains.verifyAuth(domain.id)

if (verification?.overallStatus === 'verified') {
  console.log('✅ Domain fully verified!')
  console.log('✅ DKIM:', verification.sesStatus.dkimVerified)
  console.log('✅ SPF:', verification.dnsRecords.find(r => r.name.includes('spf'))?.isVerified)
} else {
  console.log('⏳ Verification pending')
  console.log('Next steps:', verification?.nextSteps)
}
```

### Build Email Newsletter

```typescript
// Fetch logo and convert to base64
const logoResponse = await fetch('https://yourdomain.com/logo.png')
const logoBuffer = await logoResponse.arrayBuffer()
const logoBase64 = Buffer.from(logoBuffer).toString('base64')

// Send newsletter with embedded logo
const { data, error } = await inbound.emails.send({
  from: 'newsletter@yourdomain.com',
  to: ['subscriber1@example.com', 'subscriber2@example.com'],
  subject: 'Monthly Newsletter - December 2024',
  html: `
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
      </head>
      <body style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
        <header style="text-align: center; padding: 40px; background: #f8f9fa;">
          <img src="cid:company-logo" alt="Company Logo" style="max-width: 200px;" />
          <h1 style="color: #333; margin: 20px 0;">December Newsletter</h1>
        </header>
        
        <main style="padding: 40px;">
          <h2>What's New This Month</h2>
          <p>Discover our latest features and updates...</p>
        </main>
        
        <footer style="text-align: center; padding: 20px; background: #333; color: white;">
          <p>© 2024 Your Company. All rights reserved.</p>
        </footer>
      </body>
    </html>
  `,
  text: 'December Newsletter - Discover our latest features...',
  attachments: [
    {
      content: logoBase64,
      filename: 'logo.png',
      contentType: 'image/png',
      content_id: 'company-logo'
    }
  ],
  tags: [
    { name: 'campaign', value: 'newsletter' },
    { name: 'month', value: 'december' }
  ]
})
```

## SDK Reference

### Configuration

```typescript
interface InboundConfig {
  apiKey: string
  baseUrl?: string // Default: 'https://inbound.new'
  timeout?: number // Request timeout in ms
}

const inbound = new Inbound('your_api_key', {
  baseUrl: 'https://custom-domain.com',
  timeout: 30000
})
```

### Response Format

All SDK methods return a standardized response:

```typescript
interface SDKResponse<T> {
  data?: T
  error?: {
    message: string
    code?: number
    details?: any
  }
}
```

### Resource Methods

#### `inbound.emails`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `send(params)` | `POST /api/v2/emails` | Send an email |
| `get(id)` | `GET /api/v2/emails/{id}` | Get sent email |
| `reply(id, params)` | `POST /api/v2/emails/{id}/reply-new` | Reply to email or thread |
| `schedule(params)` | `POST /api/v2/emails/schedule` | Schedule email |
| `getScheduled(id)` | `GET /api/v2/emails/schedule/{id}` | Get scheduled email |
| `cancelScheduled(id)` | `DELETE /api/v2/emails/schedule/{id}` | Cancel scheduled email |
| `listScheduled(params)` | `GET /api/v2/emails/schedule` | List scheduled emails |

#### `inbound.threads`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `list(params)` | `GET /api/v2/threads` | List threads |
| `get(id)` | `GET /api/v2/threads/{id}` | Get thread details |
| `action(id, params)` | `POST /api/v2/threads/{id}/actions` | Perform thread action |
| `stats()` | `GET /api/v2/threads/stats` | Get thread statistics |

#### `inbound.domains`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `create(params)` | `POST /api/v2/domains` | Add domain |
| `list(params)` | `GET /api/v2/domains` | List domains |
| `get(id, params)` | `GET /api/v2/domains/{id}` | Get domain details |
| `update(id, params)` | `PUT /api/v2/domains/{id}` | Update domain |
| `delete(id)` | `DELETE /api/v2/domains/{id}` | Delete domain |
| `initAuth(id, params)` | `POST /api/v2/domains/{id}/auth` | Initialize authentication |
| `verifyAuth(id)` | `PATCH /api/v2/domains/{id}/auth` | Verify authentication |
| `getDnsRecords(id)` | `GET /api/v2/domains/{id}/dns-records` | Get DNS records |

#### `inbound.emailAddresses`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `create(params)` | `POST /api/v2/email-addresses` | Create email address |
| `list(params)` | `GET /api/v2/email-addresses` | List email addresses |
| `get(id)` | `GET /api/v2/email-addresses/{id}` | Get email address |
| `update(id, params)` | `PUT /api/v2/email-addresses/{id}` | Update email address |
| `delete(id)` | `DELETE /api/v2/email-addresses/{id}` | Delete email address |

#### `inbound.endpoints`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `create(params)` | `POST /api/v2/endpoints` | Create endpoint |
| `list(params)` | `GET /api/v2/endpoints` | List endpoints |
| `get(id)` | `GET /api/v2/endpoints/{id}` | Get endpoint details |
| `update(id, params)` | `PUT /api/v2/endpoints/{id}` | Update endpoint |
| `delete(id)` | `DELETE /api/v2/endpoints/{id}` | Delete endpoint |
| `test(id, params)` | `POST /api/v2/endpoints/{id}/test` | Test endpoint |

## Best Practices

<AccordionGroup>
<Accordion title="Use Idempotency Keys for Critical Emails">
Always use idempotency keys for important transactional emails to prevent duplicates:

```typescript
await inbound.emails.send(
  emailParams,
  { headers: { 'Idempotency-Key': `order-${orderId}` } }
)
```
</Accordion>

<Accordion title="Implement Webhook Signature Verification">
Verify webhook requests are from Inbound:

```typescript
import { verifyWebhookSignature } from 'inboundemail'

const isValid = verifyWebhookSignature(
  req.body,
  req.headers['x-inbound-signature'],
  process.env.WEBHOOK_SECRET
)

if (!isValid) {
  return res.status(401).json({ error: 'Invalid signature' })
}
```
</Accordion>

<Accordion title="Handle Attachments Efficiently">
For large attachments, use remote URLs instead of base64 to reduce payload size:

```typescript
// Efficient: Reference remote file
attachments: [
  {
    path: 'https://cdn.yourdomain.com/files/invoice.pdf',
    filename: 'invoice.pdf'
  }
]

// Less efficient: Large base64 payload
attachments: [
  {
    content: veryLargeBase64String,
    filename: 'invoice.pdf'
  }
]
```
</Accordion>

<Accordion title="Use Threading for Better UX">
Enable threading for support and notification emails:

```typescript
// Original notification
const { data: original } = await inbound.emails.send({
  from: 'notifications@yourdomain.com',
  to: 'user@example.com',
  subject: 'Order #123 Shipped',
  text: 'Your order has shipped!'
})

// Follow-up maintains thread
const { data: followUp } = await inbound.emails.reply(original.id, {
  from: 'notifications@yourdomain.com',
  text: 'Your order has been delivered!',
  includeOriginal: false
})
```
</Accordion>

<Accordion title="Optimize Domain Verification">
Set up authentication immediately after creating a domain:

```typescript
const { data: domain } = await inbound.domains.create({
  domain: 'yourdomain.com'
})

// Initialize auth right away
await inbound.domains.initAuth(domain.id, {
  generateSpf: true,
  generateDmarc: true
})

// Poll for verification
const checkVerification = async () => {
  const { data } = await inbound.domains.verifyAuth(domain.id)
  return data?.overallStatus === 'verified'
}

// Check every 30 seconds
const interval = setInterval(async () => {
  if (await checkVerification()) {
    console.log('✅ Domain verified!')
    clearInterval(interval)
  }
}, 30000)
```
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
<Accordion title="Email not sending">
**Common causes:**
- Domain not verified
- Invalid sender email
- Rate limit exceeded
- AWS SES not configured (contact support)

**Solution:**
```typescript
// Check domain status
const { data: domain } = await inbound.domains.get('domain_id', { check: true })

if (domain?.status !== 'verified') {
  console.log('Domain not verified. Status:', domain?.status)
  console.log('Next steps:', domain?.verificationCheck?.nextSteps)
}
```
</Accordion>

<Accordion title="Webhook not receiving emails">
**Common causes:**
- Endpoint not active
- Email address not configured
- Webhook URL unreachable
- Receipt rules not configured

**Solution:**
```typescript
// Test webhook endpoint
const { data: test } = await inbound.endpoints.test('endpoint_id')

if (!test?.success) {
  console.log('Webhook test failed:', test?.error)
  console.log('Response time:', test?.responseTime)
  console.log('Status code:', test?.statusCode)
}

// Verify email address routing
const { data: address } = await inbound.emailAddresses.get('address_id')

console.log('Routing type:', address?.routing.type)
console.log('Endpoint active:', address?.routing.isActive)
console.log('Receipt rule configured:', address?.isReceiptRuleConfigured)
```
</Accordion>

<Accordion title="Attachments failing">
**Common causes:**
- File too large (>25MB per file, >40MB total)
- Blocked file type (.exe, .bat, etc.)
- Invalid base64 encoding
- Remote URL inaccessible

**Solution:**
Check attachment constraints and validate content:

```typescript
// Validate attachment size before sending
const maxSize = 25 * 1024 * 1024 // 25MB
const fileSize = Buffer.from(base64Content, 'base64').length

if (fileSize > maxSize) {
  console.error('File too large:', fileSize, 'bytes')
}
```
</Accordion>
</AccordionGroup>

## Support

<CardGroup cols={2}>
<Card title="Documentation" icon="book" href="/api-reference">
Complete API reference with all endpoints
</Card>

<Card title="GitHub" icon="github" href="https://github.com/inbound-org/inboundemail">
SDK source code and examples
</Card>

<Card title="Discord Community" icon="discord" href="https://discord.gg/inbound">
Get help from the community
</Card>

<Card title="Email Support" icon="envelope" href="mailto:support@inbound.new">
Contact our support team
</Card>
</CardGroup>

## Related Documentation

- [API Reference](/api-reference)
- [Webhook Guide](/webhook)
- [Email Threading](/sdk/threads)
- [Domain Verification](/sdk/domains)
- [Attachment Handling](/sdk/attachments)
