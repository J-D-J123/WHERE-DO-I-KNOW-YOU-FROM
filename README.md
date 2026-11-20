<a id="readme-top"></a>

<!-- PROJECT SHIELDS -->
[![Stars][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![Manifest V3][mv3-shield]][mv3-url]
[![Zero Servers][servers-shield]][privacy-url]

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM">
    <img src="confused_no_bg.png" alt="confused face scratching its head" width="140">
  </a>

  <h3 align="center">Where Do I Know You From?</h3>

  <p align="center">
    Remembers everyone at a company who has ever emailed you.
    <br />
    <a href="#nerdy-tech"><strong>See how it works »</strong></a>
    <br />
    <br />
    <a href="https://github.com/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM/issues/new?labels=bug">Report Bug</a>
    &middot;
    <a href="https://github.com/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM/issues/new?labels=enhancement">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#about-the-project">About The Project</a>
      <ul><li><a href="#built-with">Built With</a></li></ul>
    </li>
    <li><a href="#privacy">Privacy</a></li>
    <li><a href="#nerdy-tech">Nerdy Tech</a></li>
    <li><a href="#getting-started">Getting Started</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About The Project

You're emailing Mike at Example Co. You forgot that Karen from Example Co. reached out two years ago. This extension remembers for you.

Inboxes are great at finding an email when you already know what you're looking for. They aren't built to tell you, in the moment, who else you already know at a company.

Searching "Mike" returns every Mike, and searching the domain only helps if you remember to do it. Meanwhile, the recruiter or old classmate from that company is buried under marketing emails. This extension surfaces them automatically whenever you open or write an email.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Built With

* [![JavaScript][JS-badge]][JS-url]
* [![Chrome][Chrome-badge]][Chrome-url]
* [![Firefox][Firefox-badge]][Firefox-url]
* [![Gmail API][Gmail-badge]][Gmail-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- PRIVACY -->
## Privacy

- **No servers.** No backend, no analytics, no cloud. Your browser talks directly to Gmail and nothing else.
- **Headers only.** Reads `From`, `To`, `Cc`, and `Date`. Never message bodies or attachments.
- **Encrypted DB.** Every record is encrypted before being saved, and even the lookup keys are hashed so they don't reveal company names.
- **Auto-lock.** The key lives in memory only and locks after inactivity.
- **Encrypted backups.** Exports stay encrypted and can only be restored with your passphrase.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- NERDY TECH -->
## Nerdy Tech

The extension reads only who emailed you and when, never what they said. Those people get sorted by company and stored in a locked vault on your own computer that only your password can open. No one else, including me, ever sees any of your data.

Built on Manifest V3 in vanilla JavaScript, with zero dependencies and no build step. The same codebase loads in both Chrome and Firefox.

**Getting the data.** The first sync pulls message headers through the Gmail API using the restricted `gmail.metadata` scope, which can't read message bodies at all. Requests run through a bounded worker pool with exponential backoff on rate limits. Progress is checkpointed every 500 messages, so an interrupted sync can resume, and later syncs only fetch new mail. On top of that, a debounced `MutationObserver` watches the Gmail page directly, picking up new senders as they appear and detecting whether you're reading a thread or writing an email.

**Making sense of the data.** Each address is reduced to a root domain, handling subdomains and multi-part TLDs, so `karen@mail.example.com` lands under `example.com`, and `karen@mail.example.co.uk` lands under `example.co.uk`. Free-mail providers stay ungrouped, so all of @gmail.com doesn't turn into one giant "company."

**Locking the data down.** Your passphrase goes through PBKDF2-SHA256 at 600,000 iterations to produce 512 bits, which are split into two keys. The first encrypts every record with AES-256-GCM, using a fresh 12-byte IV on every write. Each record's ID is bound in as authenticated data, so rows can't be swapped or tampered with without detection. The second key drives an HMAC-SHA256 index, which gives `O(1)` domain lookups without ever storing a plaintext domain. Because the hash is keyed, someone holding the database can't just hash a list of popular domains to see which ones you have. Everything uses the browser's native Web Crypto API, with no third-party crypto libraries.

**Storing the data.** Encrypted records live in IndexedDB. While unlocked, a decrypted copy is kept in memory, so search and lookups are instant. Locking, whether manual, from the inactivity timer, or when you close the browser, wipes the key and that in-memory copy.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- GETTING STARTED -->
## Getting Started

### Prerequisites

* Chrome 121+ or Firefox 121+
* A Google Cloud project with the **Gmail API** enabled and an OAuth consent screen in **Testing** mode, with your Gmail added as a test user

### Installation

1. Clone the repo
   ```sh
   git clone https://github.com/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM.git
   ```
2. Create an OAuth client in Google Cloud
   * **Chrome:** type *Chrome Extension*, using your extension ID
   * **Firefox:** type *Web application*, with the redirect URI shown in the extension's Settings
3. Add your client ID to `manifest.json`
   ```json
   "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
   "web_client_id": "YOUR_WEB_CLIENT_ID.apps.googleusercontent.com"
   ```
4. Load the extension
   * **Chrome:** `chrome://extensions` → Developer mode → **Load unpacked**
   * **Firefox:** `about:debugging` → This Firefox → **Load Temporary Add-on** → pick `manifest.json`
5. Click the toolbar icon, create a passphrase, and press **Start first sync**.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- MARKDOWN LINKS & IMAGES -->
[stars-shield]: https://img.shields.io/github/stars/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM.svg?style=for-the-badge
[stars-url]: https://github.com/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM/stargazers
[issues-shield]: https://img.shields.io/github/issues/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM.svg?style=for-the-badge
[issues-url]: https://github.com/J-D-J123/WHERE-DO-I-KNOW-YOU-FROM/issues
[mv3-shield]: https://img.shields.io/badge/Manifest-V3-3a3a3a?style=for-the-badge
[mv3-url]: https://developer.chrome.com/docs/extensions/develop/migrate/what-is-mv3
[servers-shield]: https://img.shields.io/badge/servers-zero-fde068?style=for-the-badge
[privacy-url]: #privacy
[JS-badge]: https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black
[JS-url]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[Chrome-badge]: https://img.shields.io/badge/Chrome-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white
[Chrome-url]: https://developer.chrome.com/docs/extensions
[Firefox-badge]: https://img.shields.io/badge/Firefox-FF7139?style=for-the-badge&logo=firefoxbrowser&logoColor=white
[Firefox-url]: https://extensionworkshop.com
[Gmail-badge]: https://img.shields.io/badge/Gmail%20API-EA4335?style=for-the-badge&logo=gmail&logoColor=white
[Gmail-url]: https://developers.google.com/gmail/api
