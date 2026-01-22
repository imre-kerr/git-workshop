---
theme: gaia
_class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('https://marp.app/assets/hero-background.svg')
marp: true
style: |
    section.dense {
        p, pre, ul, ol {
            font-size: 0.8rem;
            margin-top: 0.3rem;
            margin-bottom: 0.3rem
        }
    }
---

# Git for (litt) viderekomne

*En praktisk guide til å lage rot og fikse opp i det*

Imre Kerr

<!--
## Hvordan bli en git-guru

- Inngående kunnskap om hvordan ting faktisk henger sammen
- Hva *er* en commit, branch, ff vs 3way merge, etc

Men å lære abstrakte ting er vanskelig uten knagger å henge på.
Derfor holder vi det praktisk for nå.
-->

---

## Recap av basics

* Start: `git init`
* Lagre arbeidet ditt: `git add` og `git commit`
* Jobb med remotes: `git push` og `git pull`
* Branching: `git switch` og `git merge`

<!--

init: gjøres én gang, men må skje for at en mappe skal være et git-repo

add/commit: 
    Nybegynnere lurer ofte på hvorfor dette er separate kommandoer.
    Dette er fordi man ikke alltid har lyst til å ta med alle endringer i en commit.

push/pull:
    Igjen: "Hvorfor må jeg pushe? Skal ikke greiene mine være lagret i git når jeg kjører commit?"
    Blander git med github. Git er desentralisert, og github (og stash, gitlab etc) er bare tjenester som kjører git.
    Kunne også pushet til og pullet fra laptopen til sidemannen (gitt ssh-tilgang)

Livet uten branching: Sånn brukte vi git første gangen jeg tok i det (2012?)
Merge-konflikter hele tida, utkommentert brukket kode fordi ting ikke var klare ennå...
Bruk branches. De er hele grunnen til at Git erstattet eldre versjonskontrollsystemer.
--->

---

## Commit ofte

- Så fort noe er commitet, er det så å si umulig å miste det.
- Bedre å lage en rotete historikk og så rydde opp i etterkant.

<!-- - Så fort noe er commitet, er det lagret i git, og er nesten umulig å miste helt.
- Uansett hvor rotete ting blir, kan noen hjelpe deg hvis du har fulgt denne ene regelen.

NB! Garbage collection! -->

---

## Reachability

En commit er reachable, dersom den:

a) pekes på av en ref (branch eller tag), eller  
b) er forelderen til en annen reachable commit

**Regel:** Reachable commits blir aldri slettet automatisk!

<!-- Tilgjengelig? Nåbar? -->

---

## Lag en midlertidig branch før du gjør noe skummelt

```bash
git branch save-point-1
```

Hvis branchen din blir ødelagt:

```bash
git reset --hard save-point-1
```

…og alt er som før.

<!-- Skummelt: Omskriving av historikk, store merges

**Brancher er bare pekere til commits!** -->

---
<!-- _class: dense -->

## Omskriving av historikk

Rette på siste commit:

```bash
git commit --amend
```

Flytte aktiv branch til en annen commit:

```bash
git reset [--soft|--mixed|--hard] <commit>
```

Kopiere commit til aktiv branch (ikke omskriving, teknisk sett):

```bash
git cherry-pick <commit>
```

Ultimat multiverktøy:

```bash
git rebase --interactive [--onto <newbase>] <upstream> [<branch>]
```

<!-- Lett å skape problemer her…

Reset uten args skriver ikke om historikk.

`git rebase --interactive` kan:

- Slette enkeltcommits
- Slå sammen commits
- Splitte opp commits (bruk “edit”!)
- Endre rekkefølge på commits
- Rette på commits lengre bak i historikken 

Anbefaler å lese docs på rebase. Den er omfattende og forklarer godt.
-->

---
<!-- _class: lead --->

# Ikke skriv om historikk på delte brancher!

<!-- I praksis: Ikke force-push main/master, dev/stage/prod, versjonsbrancher etc.

Kan håndheves med branch protection på GitHub. -->

---

## Ryddig commit-historikk

- Letter code review, bisect, lettere å finne når og hvorfor endringer ble gjort…
- Flytt rundt på commits, slå sammen, splitt opp

Splitte ucommitede endringer:

```bash
git add --patch
```

---
<!-- _class: dense -->

## Bisect – finn når en bug ble introdusert

```bash
git bisect start
git bisect bad
git bisect good <kjent fungerende commit>
```

Deretter, til git sier at du er ferdig:

```bash
git bisect [good|bad]
```

Eller automatisk testing:

```bash
git bisect run <script> [<args>]
```

Best om alle commits bygger og tester grønt (annet enn den aktuelle bugen)

Alternativt:

```bash
git bisect skip
```

---
<!-- _class: dense -->

![bg right:40% vertical fit](./images/log.jpg)
![bg fit](./images/log_with_reflog.jpg)


## Finne unreachable commits

Historikk over commits du har vært innom:

```bash
git reflog
# eller:
git reflog <ref>
```

Finn hashen til commiten din, og lag en branch som peker på den:

```bash
git branch lost-commit a1b2c3
```

Vanskelig å lese output fra `git reflog`?

```bash
git log --reflog --graph --oneline
```

<!-- Når du har gjort dette kan du gjøre merge, cherry-pick, reset, restore, rebase eller hva du måtte trenge for å få ting på stell igjen.

Reflog med ref: "Hvordan har denne reffen flyttet på seg?"
Uten: Viser alle

Siste kommandoen lager en commit-graf som ligner på den man kjenner fra GUI-verktøy.

Kan GUI-verktøyet ditt gjøre det kanskje? (Seriøst, kan det? For jeg vet ikke.) -->

---

![bg right:25% fit](./images/github_create_branch.jpg)


## Finne igjen unreachable commits… på GitHub

“Repository events” – GitHubs “reflog”

```text
https://api.github.com/repos/{owner}/{repo}/events
```

Finn event av type `PushEvent`, sjekk `before`-feltet.

```text
https://github.com/{owner}/{repo}/tree/{hash}
```

…og lag ny branch fra dropdown. 

<!-- Events-APIet returnerer all mulig drit. Her må du sikkert grave litt.

Bedre å unngå det helt, med branch protection

Pull requests viser force-pushes…

Alternativt kan du spore opp en utvikler som har commiten på sin maskin og be dem om hjelp. -->

---

## Finne commit i historikken

`git log` har en søkefunksjon:

```bash
git log -S <string>
```

Kan såklart kombineres med `--reflog`.

---
<!-- _class: dense -->

## Fjerne ting fra git-historikken

Hemmeligheter, GDPR-brudd, store filer…

På feature-branch:

- `git commit --amend`
- `git rebase --interactive`

Lengre tilbake: BFG Repo-Cleaner

```text
https://rtyley.github.io/bfg-repo-cleaner/
```

:warning: For å slette ting permanent etterpå:

```bash
git reflog expire --expire=now --all && git gc --prune=now --aggressive
```

:x: Ikke bruk: `git filter-branch`

<!-- Ukrypterte passord skal ikke sjekkes inn… Finnes verktøy for å finne dem i historikken:

```text
https://thecyberpunker.com/blog/git-exposed-pentesting-git-tools/
```

Testen på hvilken du skal bruke: Er commiten(e) bare reachable fra én ref? I så fall kan du bruke de innebygde verktøyene.

NBNB! Hva ligger på utviklermaskinene?

Filter-branch er så crap at selv dokumentasjonen til kommandoen anbefaler at du bruker noe annet. -->

---
<!-- _class: dense -->

## BFG Repo-Cleaner, steg for steg

Alt må renses. Derfor:

**Alle på teamet:**
1. Commit og push (feature/midlertidig branch er fint)

**En person:**
2. `git fetch`, og så `git switch {branch}` for hver remote-branch
3. Kjør BFG  
4. `git push --force --all`  
5. `git push --force --tags`

**Alle:**
6. Slett repo-mappa og clone på nytt.

---

## Videre lesning

Mitt :bulb:-øyeblikk:

```text
http://think-like-a-git.net/
```

Dypdykk:

```text
https://github.com/pluralsight/git-internals-pdf
https://git-scm.com/book/en/v2
```

---

## Tid for oppgaver!

Last ned oppgaver og slides:

```text
https://bit.ly/3wCkAh4
```

- Jobb gjerne i par/små grupper
- Flere av oppgavene kan løses på flere måter. Diskuter pros/cons.
