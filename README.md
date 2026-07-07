name: Actualizar README

on:
  schedule:
    - cron: '0 6 * * *'   # todos los días a las 6:00 UTC (1:00 am Lima)
  workflow_dispatch:       # también puedes ejecutarlo manualmente desde la pestaña Actions
  push:
    branches: [ main, master ]

jobs:
  actividad-reciente:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: master

      - name: Obtener y escribir actividad reciente
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GH_USERNAME: JeanHaro
        run: |
          python3 << 'EOF'
          import json, os, re, urllib.request

          token = os.environ["GH_TOKEN"]
          username = os.environ["GH_USERNAME"]

          url = f"https://api.github.com/users/{username}/events/public?per_page=30"
          req = urllib.request.Request(url, headers={
              "Authorization": f"Bearer {token}",
              "User-Agent": "readme-bot",
              "Accept": "application/vnd.github+json"
          })
          events = json.load(urllib.request.urlopen(req))

          lines = []
          for e in events:
              if len(lines) >= 5:
                  break
              t = e["type"]
              repo = e["repo"]["name"]
              if t == "PushEvent":
                  n = len(e["payload"].get("commits", []))
                  lines.append(f"🚀 Push de {n} commit(s) a **{repo}**")
              elif t == "CreateEvent" and e["payload"].get("ref_type") == "repository":
                  lines.append(f"✨ Creó el repositorio **{repo}**")
              elif t == "PullRequestEvent":
                  lines.append(f"🔀 {e['payload']['action']} un PR en **{repo}**")
              elif t == "IssuesEvent":
                  lines.append(f"📋 {e['payload']['action']} un issue en **{repo}**")

          body = "\n".join(f"- {l}" for l in lines) if lines else "- Sin actividad pública reciente"
          block = f"<!--START_SECTION:activity-->
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
<!--END_SECTION:activity-->"

          with open("README.md", encoding="utf-8") as f:
              content = f.read()

          content = re.sub(r"<!--START_SECTION:activity-->
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
- 🚀 Push de 0 commit(s) a **JeanHaro/JeanHaro**
<!--END_SECTION:activity-->", block, content, flags=re.S)

          with open("README.md", "w", encoding="utf-8") as f:
              f.write(content)
          EOF

      - name: Commitear cambios
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git diff --staged --quiet || git commit -m "docs: actualizar actividad reciente"
          git push

  tiempo-programando:
    runs-on: ubuntu-latest
    needs: actividad-reciente
    if: always()
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: master

      - name: Calcular tiempo programando
        run: |
          START_DATE="2020-10-01"
          NOW=$(date +%s)
          START=$(date -d "$START_DATE" +%s)
          DIFF_DAYS=$(( (NOW - START) / 86400 ))
          YEARS=$(( DIFF_DAYS / 365 ))
          MONTHS=$(( (DIFF_DAYS % 365) / 30 ))
          TEXT="⏱️ programando desde hace ${YEARS} años y ${MONTHS} meses"
          sed -i "s|<!--START_SECTION:coding-time--> ⏱️ programando desde hace 5 años y 9 meses <!--END_SECTION:coding-time-->|" README.md

      - name: Commitear cambios
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git diff --staged --quiet || git commit -m "docs: actualizar tiempo programando"
          git push

  snake:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Generar snake de contribuciones
        uses: Platane/snk@v3
        with:
          github_user_name: JeanHaro
          outputs: |
            dist/github-contribution-grid-snake.svg
            dist/github-contribution-grid-snake-dark.svg?palette=github-dark

      - name: Publicar en la rama output
        uses: crazy-max/ghaction-github-pages@v4
        with:
          target_branch: output
          build_dir: dist
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
