# WHERE-DO-I-KNOW-YOU-FROM
You’re emailing Mike at Example Co. You forgot that Karen from Example Co. reached out two years ago. This extension remembers for you.

## Reason. 
Inboxes are great at finding an email when you already know what you're looking for. They aren't built to tell you, in the moment, who else you already know at a company.

Searching "Mike" returns every Mike, and searching the domain only helps if you remember to do it. Meanwhile, the recruiter or old classmate from that company is buried under marketing emails. This extension surfaces them automatically whenever you open or write an email.

## Privacy. 

- **No servers** No backend, no analytics, no cloud. Your browser talks directly to Gmail and nothing else.
- **Headers only.** Reads `From`, `To`, `Cc`, and `Date`. Never message bodies or attachments.
- **Encrypted DB** Every record is encrypted before it's saved, and even the lookup keys are hashed so they don't reveal company names.
- **Encrypted DB**
- **Auto-lock.** The key lives in memory only and locks after inactivity.
- **Encrypted backups** Exports stay encrypted and can only be restored with your password/ passphrase

## Nerdy Tech. 

The extension reads only who emailed you and when, never what they said. Those people get sorted by company and stored in a locked vault on your own computer that only your password can open. No one else, including me, ever sees any of your data.

Built on Manifest V3 in vanilla JavaScript, with zero dependencies and no build step. The same codebase loads in both Chrome and Firefox.

**Getting the data.** The first sync pulls message headers through the Gmail API using the restricted `gmail.metadata` scope, which can't read message bodies at all. Requests run through a bounded worker pool with exponential backoff on rate limits. Progress is checkpointed every 500 messages, so an interrupted sync can resume, and later syncs only fetch new mail. On top of that, a debounced `MutationObserver` watches the Gmail page directly, picking up new senders as they appear and detecting whether you're reading a thread or writing an email.

**Making sense of the data.** Each address is reduced to a root domain, handling subdomains and multi-part TLDs, so `karen@mail.acme.co.uk` lands under `acme.co.uk`. Free-mail providers stay ungrouped, so all of @gmail.com doesn't turn into one giant "company."

**Locking the data down.** Your passphrase goes through PBKDF2-SHA256 at 600,000 iterations to produce 512 bits, which are split into two keys. The first encrypts every record with AES-256-GCM, using a fresh 12-byte IV on every write. Each record's ID is bound in as authenticated data, so rows can't be swapped or tampered with without detection. The second key drives an HMAC-SHA256 index, which gives `O(1)` domain lookups without ever storing a plaintext domain. Because the hash is keyed, someone holding the database can't just hash a list of popular domains to see which ones you have. Everything uses the browser's native Web Crypto API, with no third-party crypto libraries.

**Storing the data.** Encrypted records live in IndexedDB. While unlocked, a decrypted copy is kept in memory, so search and lookups are instant. Locking, whether manual, from the inactivity timer, or when you close the browser, wipes the key and that in-memory copy.
