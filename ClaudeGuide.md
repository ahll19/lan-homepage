# Homepage config — how it fits together

## Which file does what

| File                                               | Controls                                                                                    |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `settings.yaml`                                    | Global theme, colors, and the **layout** — how groups are arranged on the page              |
| `widgets.yaml`                                     | The header bar only — date/time, search, weather. Not tied to any service.                  |
| `services.yaml`                                    | The actual dashboard body — your groups and service cards                                   |
| `bookmarks.yaml`                                   | Optional secondary list of plain links (no widgets), usually shown in a sidebar             |
| `.env`                                             | Secrets (API keys/tokens) kept out of the yaml files                                        |
| `docker.yaml` / `kubernetes.yaml` / `proxmox.yaml` | Only needed if you turn on auto-discovery for those backends — you're skipping this for now |

## The core mental model

There are **two different kinds of widget** and it's the most common point of confusion:

1. **Info widgets** (`widgets.yaml`) — standalone, live in the header bar. Weather, search, clock. Not connected to any app.
2. **Service widgets** (`services.yaml`, nested under a service) — pull live stats from one specific app's API (e.g. Plex "now playing", TrueNAS disk usage). These appear *inside* that service's card.

So "TrueNAS resource stats" isn't a header widget — it's a `widget:` block nested under your TrueNAS entry in `services.yaml`, using `type: truenas`.

## Anatomy of one service entry

```yaml
- GroupName:
    - Display Name:
        icon: something.png       # see Icons below
        href: https://...         # where the card links when clicked
        description: subtext      # small grey line under the name
        widget:                   # OPTIONAL — omit entirely for a plain bookmark-style card
          type: sometype           # which integration (see below)
          url: http://internal-ip:port
          key: "{{HOMEPAGE_VAR_X}}"
```

**`href` vs widget `url`**: `href` is what your browser opens when you click (your NPM HTTPS hostname). Widget `url` is what the Homepage *server* calls to fetch stats — since Homepage runs on your network already, you can usually point this straight at the internal `http://ip:port` and skip the reverse proxy hop entirely. Both are independent; set them differently on purpose.

## Adding a new app yourself — step by step

1. Pick which **group** it belongs in (Media / Home / Infrastructure / Apps), or add a new group name.
2. Add an entry under that group in `services.yaml` with just `icon`, `href`, `description` first — get it showing as a plain link before attempting a widget.
3. If it's a new group name, add a matching key under `layout:` in `settings.yaml` — the names must match exactly (case-sensitive) or your group won't render correctly.
4. Check if it has a **widget integration**: full list at https://gethomepage.dev/widgets/services/ — search the page for your app's name. Each entry there tells you exactly which fields (`type`, `url`, plus any auth field) it needs.
5. If it needs a key/token, generate it from that app's settings, put it in `.env` as `HOMEPAGE_VAR_YOURNAME=xxxx`, and reference it in the yaml as `"{{HOMEPAGE_VAR_YOURNAME}}"`.
6. Restart/reload Homepage (it usually hot-reloads on config save, otherwise restart the container).

## Icons

Homepage ships with the [dashboard-icons](https://github.com/walkxcode/dashboard-icons) set built in — for most self-hosted apps, just `icon: appname.png` (or `.svg`) works if the name matches. Browse available names at https://homepage.gethomepage.dev/configs/icons/ or the icon repo directly. You can also use:
- `mdi-icon-name` for Material Design Icons
- `si-icon-name` for Simple Icons
- a direct URL or local path to your own image

## Layout tips (settings.yaml)

- Each top-level group defaults to a **column**. Up to 4 columns sit side by side automatically.
- Set `style: row` + `columns: N` on a group to make its cards flow horizontally and wrap at N per row instead — this is what the starter `settings.yaml` uses for all four groups.
- Groups can be nested for more complex layouts, but don't reach for that until the simple version feels limiting.

## Secrets

Don't hardcode real API keys directly into the yaml files sitting on an SMB share. Copy `.env.example` → `.env` in the same folder, fill in real values there, and reference them in yaml as `"{{HOMEPAGE_VAR_NAME}}"`. If your TrueNAS app deployment doesn't support mounting a `.env` file, set the same variables as environment variables on the container instead — functionally identical.

## Docs to keep open while you work

- Services + widgets: https://gethomepage.dev/configs/services/
- Full service widget list (search for your app): https://gethomepage.dev/widgets/services/
- Info widgets: https://gethomepage.dev/configs/info-widgets/
- Layout/settings: https://gethomepage.dev/configs/settings/