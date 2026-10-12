# Recast Studio

Turn one video take into a new scene using AI models on Replicate. Record yourself, upload
a clip or drag one onto the page, choose **Automatic** or describe the result you want, and
the app plans the scenario, picks the models, and generates the video.

## What it can change

| Choice | What stays | How it's made |
|---|---|---|
| Everything | Your motion and timing | Recast: a new keyframe animated with your motion (or Edit/Restyle) |
| Background only | Your face, hair, clothes and motion | Up to 10 s: AI blend (Wan 2.7 VideoEdit, relights the scene). Up to 30 s: exact cut-out, your original pixels keyed onto a new background in the browser |
| Outfit only | Your face, hair, motion and the scene | Up to 10 s: AI blend. Longer clips use Recast, where your face can shift slightly |
| Background and outfit | Your face, hair and motion | Same as Outfit only |

You approve a preview image (the new background, or you in the new outfit) before the video is made.
The exact cut-out runs the matting model twice (a green-screen pass and an alpha-mask pass, about
$0.10 together), so hair and edges blend cleanly with no green fringe. It renders in real time in
your browser, so keep the tab open and in front while it runs.

Not sure what to ask for? **Example ideas** in the header lists ready-made backgrounds, outfits
and full scenes; clicking one fills in the description and picks the matching choice.

## Videos it accepts

| | |
|---|---|
| Format | MP4 or MOV |
| Length | 3 to 30 seconds |
| Size | up to 100 MB |

**Record** uses the computer's camera and microphone, shows a 3-second countdown, and stops by
itself at 30 seconds. Chrome, Edge and Safari save recordings as MP4. Browsers that can only
record WebM still work, and the app notes it in the activity log.

## App icon

The icon is built into `index.html`, so there's nothing extra to upload. It shows in the browser
tab, bookmarks and history. In Chrome or Edge, use **Install Recast Studio** (the install
icon in the address bar, or the browser menu) to open it as its own app with its own taskbar
or Dock icon. On iPhone or iPad, use **Share → Add to Home Screen**.

## How the pieces fit

| Piece | Where it runs | What it does |
|---|---|---|
| `index.html` | GitHub Pages | The whole app: video loading, keyframes, planning, generation, downloads |
| `worker.js` | Cloudflare Workers (free) | Passes the page's calls on to Replicate, because Replicate blocks direct calls from web pages |

Visitors paste their own Replicate API key. No key is stored in this repository or in the
Worker. If someone ticks **Remember in this browser**, the key stays in their browser only.

## Setup (free, about 10 minutes)

### 1. Deploy the Worker

1. Sign in at [dash.cloudflare.com](https://dash.cloudflare.com) (the free plan is enough).
2. Go to **Workers & Pages**, create a new Worker from the **Hello World** template, name it
   `recast-relay`, and deploy it.
3. Open the Worker's code editor, replace everything with the contents of `worker.js`,
   and deploy again.
4. Copy the Worker's address, for example `https://recast-relay.yourname.workers.dev`.
   Opening it in a browser should show *Recast Studio relay is running.*

### 2. Publish the page on GitHub Pages

1. Create a **public** repository (for example `recast-studio`) and upload `index.html`
   and this `README.md`.
2. Edit `index.html` on GitHub. Near the top of the `<script>` section, find:

   ```js
   const RELAY_URL = "";
   ```

   and paste your Worker address between the quotes. Commit the change.
3. Go to **Settings → Pages**, set **Source** to *Deploy from a branch*, choose `main` and
   `/ (root)`, and save.
4. After a minute or two the app is live at `https://yourname.github.io/recast-studio/`.

### 3. Lock the Worker to your page (recommended)

Out of the box the Worker accepts calls from any web page. To allow only yours, go to the
Worker's **Settings → Variables and secrets** and add a plain-text variable:

- Name: `ALLOWED_ORIGINS`
- Value: `https://yourname.github.io`

Use just the scheme and domain. Leave off the repository path and any trailing slash. Separate
more than one address with commas. You can edit the `ALLOWED_ORIGINS` line at the top of
`worker.js` instead.

## Custom domain (optional)

To serve the app from something like `recast.teachingtools.dev`:

1. In the repository's **Settings → Pages → Custom domain**, enter the domain and save.
2. At your DNS provider, add a `CNAME` record for `recast` pointing to `yourname.github.io`.
3. Add `https://recast.teachingtools.dev` to the Worker's `ALLOWED_ORIGINS`.

## Costs and limits

- **GitHub Pages**: free for public repositories.
- **Cloudflare Workers free plan**: 100,000 requests a day and uploads up to 100 MB.
  While a job runs, the page checks progress every 2 seconds, so a 3-minute generation uses
  about 100 requests.
- **Replicate**: billed to the API key used. The app shows an estimate before you generate,
  and your Replicate dashboard shows the real charges.

## Troubleshooting

- **Yellow banner saying the page can't reach the relay**: check the `RELAY_URL` address in
  `index.html`, and check that `ALLOWED_ORIGINS` lists the page's address exactly
  (for example `https://yourname.github.io`).
- **"Replicate rejected the API key"**: copy a fresh token from replicate.com under
  Account → API tokens.
- **Upload fails**: keep videos under 100 MB. Trim or compress longer clips.
- **Record says camera access was blocked**: click the camera icon in the address bar, allow the
  camera and microphone for the site, then press Record again. Recording only works on an
  `https://` address (GitHub Pages is) or on `localhost`.
- **Restyle result has no sound**: Luma Modify Video can return silent clips. The desktop
  version (`recast_studio.py`) re-attaches the original audio with ffmpeg.
- **"Video frame rate must be >= 16fps"**: webcams in dim rooms often record at 15 fps, which
  Wan VideoEdit refuses. The app now reads the frame rate when a clip loads (it shows in the
  file line, with "(low)" under 16 fps) and makes a 30 fps copy in your browser before an
  edit model sees it; if a model still refuses a clip, the copy is made and the step is retried
  once. The copy takes about as long as the clip, so keep the tab in front. Recording in a
  brighter room avoids it altogether.
- **Changed your mind after planning**: edit the instruction box before pressing Generate, or
  while the "Happy with this look?" panel is open and press "Try another look"; the next image
  uses your text. Changing the description instead makes Generate plan the shot again first.
  If a video step fails after you approved a look, the next Generate reuses that look.

## Running it on your own computer instead

Put `recast_relay.py` in the same folder as `index.html` and run `python recast_relay.py`.
It serves the page at `http://localhost:8765` and does the Worker's job on your machine.
If `RELAY_URL` is set, the page still uses the Worker, so leave it blank for a local-only copy.
