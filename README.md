# zed-config

Configuration personnelle de Zed, avec une installation sur macOS, Windows et Linux Ubuntu/Debian.

Le compte Zed ne synchronise pas ces fichiers. Le dépôt sert à les transférer ; il ne synchronise pas automatiquement les changements effectués dans Zed.

## Contenu

| Fichier | Rôle |
| --- | --- |
| `settings.json` | Apparence Fleet, polices, Python, modèles Ollama et installation automatique de JetBrains Icons |
| `keymap.json` | Raccourcis personnels, avec Cmd sur macOS |
| `themes/fleet-dark-edited.json` | Thème Fleet Dark - Edited |
| `snippets/python.json` | Snippets `main`, `mark`, `patch` et `dsp` |

Modèles utilisés :

- Autocomplétion : `qwen2.5-coder:7b-base`.
- Messages de commit : `qwen2.5-coder:7b-instruct`.
- Serveur : `http://127.0.0.1:11434`. Aucun abonnement Zed n'est nécessaire pour ces deux usages locaux.

Polices : **Inter Variable** pour l'interface et **JetBrains Mono** pour le code. L'extension **JetBrains Icons** (`jetbrains-icons`) s'installe automatiquement au lancement de Zed.

Version de référence de la configuration : Zed 1.23.2. Les commandes ci-dessous installent les versions stables proposées par les fournisseurs.

## Avant de commencer

1. Sur un ordinateur client, obtenir l'autorisation d'installer Zed, Ollama et des modèles IA. Ne pas contourner les restrictions de l'entreprise.
2. Disposer d'un compte GitHub autorisé à lire le dépôt privé `Mdhvince/zed-config`.
3. Prévoir une connexion Internet pour l'installation et environ **9,4 Go pour les modèles**, en plus des applications et des polices. La RAM et le GPU déterminent les performances ; les deux modèles 7B ne seront pas aussi rapides sur tous les PC.
4. Fermer Zed avant de copier la configuration. Les procédures sauvegardent une configuration existante **hors du dépôt**, puis remplacent seulement les quatre fichiers de configuration listés ci-dessus.
5. Suivre uniquement la section correspondant au système, dans l'ordre. En cas d'erreur, arrêter la procédure et corriger l'erreur avant de poursuivre.

Les chemins documentés sont ceux d'une installation standard. Adapter la destination si Zed utilise un répertoire de configuration personnalisé. Ne pas appliquer la copie ci-dessous à une configuration gérée par des liens symboliques ou des jonctions : utiliser son gestionnaire existant.

Les interpréteurs Python et les dépendances de chaque projet, notamment pytest ou Spark, restent à installer selon les instructions du projet.

## macOS

Utiliser Terminal avec le shell zsh fourni par macOS.

### 1. Installer Homebrew et les applications

Si Homebrew est absent :

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Configurer Homebrew dans zsh, pour Apple Silicon ou Intel, puis installer les dépendances :

```bash
if [ -x /opt/homebrew/bin/brew ]; then
  brew_env='eval "$(/opt/homebrew/bin/brew shellenv)"'
else
  brew_env='eval "$(/usr/local/bin/brew shellenv)"'
fi

touch "$HOME/.zprofile"
grep -Fxq "$brew_env" "$HOME/.zprofile" || printf '\n%s\n' "$brew_env" >> "$HOME/.zprofile"
eval "$brew_env"

brew install git gh
brew install --cask zed ollama-app font-inter font-jetbrains-mono
```

Utiliser le cask `ollama-app`, pas une seconde installation indépendante du serveur Ollama. Si une application est déjà installée, conserver une seule installation et vérifier son fonctionnement avant de continuer.

### 2. Se connecter à GitHub et cloner

La connexion ouvre le navigateur. Utiliser le compte ayant accès au dépôt privé.

```bash
(
  set -eu
  gh auth login --hostname github.com --git-protocol https --web
  mkdir -p "$HOME/personal_projects"
  gh repo clone Mdhvince/zed-config "$HOME/personal_projects/zed-config"
)
```

Si le dossier cible existe déjà, ne pas le supprimer. Utiliser la section « Mettre à jour » ou choisir un autre chemin pour le clone.

### 3. Sauvegarder et copier les réglages

Fermer Zed, puis exécuter :

```bash
(
  set -eu
  cd "$HOME/personal_projects/zed-config"
  config="$HOME/.config/zed"

  if [ -d "$config" ]; then
    backup="$(mktemp -d "${config}.backup.XXXXXX")"
    cp -a "$config/." "$backup/"
    printf 'Backup: %s\n' "$backup"
  fi

  mkdir -p "$config/themes" "$config/snippets"
  cp settings.json keymap.json "$config/"
  cp themes/fleet-dark-edited.json "$config/themes/"
  cp snippets/python.json "$config/snippets/"
)
```

### 4. Démarrer Ollama et télécharger les modèles

```bash
open -a Ollama
curl --fail --silent --show-error http://127.0.0.1:11434/api/version
```

Attendre que la commande `curl` retourne la version du serveur. Si elle échoue pendant le démarrage, la relancer avant de poursuivre.

Utiliser le CLI inclus dans l'application, pour éviter une éventuelle autre version présente dans le PATH :

```bash
(
  set -eu
  ollama_bin="/Applications/Ollama.app/Contents/Resources/ollama"
  "$ollama_bin" pull qwen2.5-coder:7b-base
  "$ollama_bin" pull qwen2.5-coder:7b-instruct
  "$ollama_bin" list
)
```

### 5. Ouvrir Zed

```bash
open -a Zed
```

Poursuivre avec la section « Vérifier l'installation ».

## Windows

Utiliser PowerShell. WinGet doit être disponible ; il est fourni par **App Installer** sur les versions Windows compatibles. Si WinGet est absent ou bloqué, demander à l'administrateur de l'installer.

### 1. Installer les applications

```powershell
winget install --exact --id Git.Git --source winget
winget install --exact --id GitHub.cli --source winget
winget install --exact --id ZedIndustries.Zed --source winget
winget install --exact --id Ollama.Ollama --source winget
```

Accepter les demandes d'installation autorisées par l'entreprise. Si un paquet est déjà présent, vérifier qu'il fonctionne avant de passer au suivant.

**Fermer puis rouvrir PowerShell** pour actualiser le PATH.

### 2. Se connecter à GitHub et cloner

```powershell
$ErrorActionPreference = "Stop"
git --version
gh --version

gh auth login --hostname github.com --git-protocol https --web
if ($LASTEXITCODE -ne 0) { throw "GitHub authentication failed" }

$repo = Join-Path $HOME "personal_projects\zed-config"
if (Test-Path -LiteralPath $repo) { throw "Clone directory already exists: $repo" }
New-Item -ItemType Directory -Force -Path (Split-Path $repo) | Out-Null
gh repo clone Mdhvince/zed-config $repo
if ($LASTEXITCODE -ne 0) { throw "Repository clone failed" }
```

### 3. Installer les polices pour l'utilisateur courant

Les commandes téléchargent les archives officielles. Elles copient les polices dans le profil utilisateur et les enregistrent dans **HKCU**, sans modifier la politique d'exécution PowerShell ni les polices des autres utilisateurs.

```powershell
$ErrorActionPreference = "Stop"
$fontWork = Join-Path $env:TEMP ("zed-fonts-" + [guid]::NewGuid().ToString("N"))
New-Item -ItemType Directory -Path $fontWork | Out-Null

Invoke-WebRequest -UseBasicParsing -Uri "https://github.com/rsms/inter/releases/download/v4.1/Inter-4.1.zip" -OutFile "$fontWork\inter.zip"
Invoke-WebRequest -UseBasicParsing -Uri "https://github.com/JetBrains/JetBrainsMono/releases/download/v2.304/JetBrainsMono-2.304.zip" -OutFile "$fontWork\mono.zip"
Expand-Archive -Path "$fontWork\inter.zip" -DestinationPath "$fontWork\inter"
Expand-Archive -Path "$fontWork\mono.zip" -DestinationPath "$fontWork\mono"

$fontDirectory = Join-Path $env:LOCALAPPDATA "Microsoft\Windows\Fonts"
$fontRegistry = "HKCU:\Software\Microsoft\Windows NT\CurrentVersion\Fonts"
New-Item -ItemType Directory -Force -Path $fontDirectory | Out-Null
if (-not (Test-Path -LiteralPath $fontRegistry)) {
    New-Item -Path $fontRegistry | Out-Null
}

$fonts = @(Get-ChildItem "$fontWork\inter" -Recurse -File -Filter "InterVariable*.ttf")
$fonts += @(Get-ChildItem "$fontWork\mono" -Recurse -File -Filter "JetBrainsMono-*.ttf")
if ($fonts.Count -lt 3) { throw "Expected font files were not found" }

foreach ($font in $fonts) {
    $destination = Join-Path $fontDirectory $font.Name
    Copy-Item -LiteralPath $font.FullName -Destination $destination -Force
    New-ItemProperty -Path $fontRegistry -Name ($font.BaseName + " (TrueType)") -PropertyType String -Value $destination -Force | Out-Null
}
```

Fermer puis rouvrir Zed après cette étape. Si les polices ne sont pas reconnues, enregistrer le travail en cours puis fermer et rouvrir la session Windows.

### 4. Sauvegarder et copier les réglages

Fermer Zed. Cette étape convertit les raccourcis `cmd-` en `ctrl-` **dans la copie installée**, sans modifier les fichiers du dépôt.

```powershell
$ErrorActionPreference = "Stop"
$repo = Join-Path $HOME "personal_projects\zed-config"
$config = Join-Path $env:APPDATA "Zed"

if (Test-Path -LiteralPath $config) {
    $backup = $config + ".backup." + [guid]::NewGuid().ToString("N")
    Copy-Item -LiteralPath $config -Destination $backup -Recurse -ErrorAction Stop
    Write-Host "Backup: $backup"
}

New-Item -ItemType Directory -Force -Path $config, "$config\themes", "$config\snippets" | Out-Null
Copy-Item -Path "$repo\settings.json", "$repo\keymap.json" -Destination $config -Force
Copy-Item "$repo\themes\fleet-dark-edited.json" -Destination "$config\themes" -Force
Copy-Item "$repo\snippets\python.json" -Destination "$config\snippets" -Force

$keymap = [System.IO.File]::ReadAllText("$config\keymap.json")
$keymap = $keymap.Replace('"cmd-', '"ctrl-')
[System.IO.File]::WriteAllText("$config\keymap.json", $keymap, [System.Text.UTF8Encoding]::new($false))
```

### 5. Vérifier Ollama et télécharger les modèles

L'installateur Ollama lance normalement le serveur en arrière-plan. Vérifier qu'il répond :

```powershell
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/version"
```

S'il ne répond pas, ouvrir l'application Ollama depuis le menu Démarrer. Autre possibilité : exécuter `ollama serve` dans un second terminal et garder ce terminal ouvert. Ne pas démarrer un second serveur si le premier fonctionne.

Une fois l'API disponible :

```powershell
ollama pull qwen2.5-coder:7b-base
if ($LASTEXITCODE -ne 0) { throw "Autocomplete model download failed" }
ollama pull qwen2.5-coder:7b-instruct
if ($LASTEXITCODE -ne 0) { throw "Commit model download failed" }
ollama list
```

### 6. Ouvrir Zed

```powershell
zed
```

Si le CLI n'est pas dans le PATH, ouvrir Zed depuis le menu Démarrer. Poursuivre avec la section « Vérifier l'installation ».

## Linux : Ubuntu / Debian

Utiliser Bash sur un système de bureau 64 bits pris en charge par Zed. Zed nécessite un GPU et des pilotes Vulkan compatibles. Faire installer les pilotes appropriés par l'administrateur si nécessaire.

### 1. Installer les dépendances et les applications

```bash
sudo apt-get update
sudo apt-get install -y git gh curl ca-certificates unzip fontconfig libvulkan1 mesa-vulkan-drivers
```

Installer Zed et Ollama avec leurs installateurs officiels :

```bash
(
  set -eu
  work="$(mktemp -d)"
  curl -fsSL https://zed.dev/install.sh -o "$work/install-zed.sh"
  sh "$work/install-zed.sh"
  curl -fsSL https://ollama.com/install.sh -o "$work/install-ollama.sh"
  sh "$work/install-ollama.sh"
)
```

Le serveur Ollama peut nécessiter des droits administrateur. Les installateurs sont téléchargés depuis leurs sites officiels ; les examiner avant exécution si la politique de l'entreprise le demande.

Rendre le CLI Zed accessible dans Bash :

```bash
path_line='export PATH="$HOME/.local/bin:$PATH"'
touch "$HOME/.bashrc"
grep -Fxq "$path_line" "$HOME/.bashrc" || printf '\n%s\n' "$path_line" >> "$HOME/.bashrc"
export PATH="$HOME/.local/bin:$PATH"
```

### 2. Installer les polices

```bash
(
  set -eu
  work="$(mktemp -d)"
  font_dir="$HOME/.local/share/fonts/zed-config"
  mkdir -p "$font_dir"

  curl -fsSL https://github.com/rsms/inter/releases/download/v4.1/Inter-4.1.zip -o "$work/inter.zip"
  curl -fsSL https://github.com/JetBrains/JetBrainsMono/releases/download/v2.304/JetBrainsMono-2.304.zip -o "$work/mono.zip"
  unzip -q "$work/inter.zip" -d "$work/inter"
  unzip -q "$work/mono.zip" -d "$work/mono"

  find "$work/inter" -type f -name 'InterVariable*.ttf' -exec cp {} "$font_dir/" \;
  find "$work/mono" -type f -name 'JetBrainsMono-*.ttf' -exec cp {} "$font_dir/" \;
  fc-cache -f "$font_dir"
  fc-match 'Inter Variable'
  fc-match 'JetBrains Mono'
)
```

Les deux dernières commandes doivent identifier les familles demandées, et non une police de remplacement.

### 3. Se connecter à GitHub et cloner

```bash
(
  set -eu
  gh auth login --hostname github.com --git-protocol https --web
  mkdir -p "$HOME/personal_projects"
  gh repo clone Mdhvince/zed-config "$HOME/personal_projects/zed-config"
)
```

### 4. Sauvegarder et copier les réglages

Fermer Zed. Comme sous Windows, les raccourcis Cmd sont convertis en Ctrl dans la copie installée.

```bash
(
  set -eu
  cd "$HOME/personal_projects/zed-config"
  config="${XDG_CONFIG_HOME:-$HOME/.config}/zed"

  if [ -d "$config" ]; then
    backup="$(mktemp -d "${config}.backup.XXXXXX")"
    cp -a "$config/." "$backup/"
    printf 'Backup: %s\n' "$backup"
  fi

  mkdir -p "$config/themes" "$config/snippets"
  cp settings.json keymap.json "$config/"
  cp themes/fleet-dark-edited.json "$config/themes/"
  cp snippets/python.json "$config/snippets/"
  sed -i 's/"cmd-/"ctrl-/g' "$config/keymap.json"
)
```

### 5. Démarrer Ollama et télécharger les modèles

Sur un système avec systemd :

```bash
sudo systemctl enable --now ollama
curl --fail --silent --show-error http://127.0.0.1:11434/api/version
```

Attendre que l'API réponde. Si systemd n'est pas disponible, lancer `ollama serve` dans un second terminal, le garder ouvert et vérifier l'API avant de continuer.

```bash
(
  set -eu
  ollama pull qwen2.5-coder:7b-base
  ollama pull qwen2.5-coder:7b-instruct
  ollama list
)
```

### 6. Ouvrir Zed

```bash
zed
```

## Vérifier l'installation

1. Laisser Zed installer **JetBrains Icons**. Vérifier le thème **Fleet Dark - Edited**, les icônes et les polices dans les réglages. Le flou et la transparence dépendent du système et de son compositeur : le rendu n'est pas garanti identique à macOS.
2. Vérifier les raccourcis : **Cmd+[ / Ctrl+[** ouvre le panneau Agent ; **Cmd+] / Ctrl+]** ouvre la barre latérale des espaces de travail ; **Cmd+Shift+T / Ctrl+Shift+T** affiche le panneau Terminal. Ils remplacent les raccourcis par défaut qui utilisent les mêmes touches.
3. Ouvrir un fichier Python et tester les snippets `main`, `mark`, `patch` et `dsp`. Ce sont des modèles éditables, sans imports automatiques. Le snippet `dsp` contient deux champs de schéma pour trois valeurs par ligne : adapter le schéma avant exécution.
4. Laisser Ollama fonctionner. Commencer à écrire du code et attendre une suggestion d'autocomplétion ; **Tab** l'accepte. Zed peut aussi télécharger les composants nécessaires au langage, notamment basedpyright pour Python.
5. Dans un dépôt avec une modification, tester **Generate Commit Message**, puis relire le texte. Cette action génère le message ; elle ne crée pas le commit. Le premier appel peut être plus lent pendant le chargement du modèle.

Les modèles ne se trouvent pas dans ce dépôt. Copier les réglages sans installer Ollama et les deux modèles ne suffit pas à activer l'IA locale. Le modèle par défaut de l'Agent n'est pas fixé par cette configuration : ne pas supposer que les autres fonctions IA utilisent également Ollama.

## Prompt et trailers des messages de commit

Zed utilise son prompt natif : message court et clair, sujet à l'impératif, corps seulement si utile. `agent.commit_message_instructions` n'est pas défini dans cette configuration. Les règles du projet sont incluses selon le comportement par défaut de Zed.

Le générateur n'impose pas de trailer IA tel que « Generated by Qwen ». Zed peut ajouter un `Co-authored-by:` pour les collaborateurs d'une session partagée. Des règles ou hooks propres au projet peuvent produire d'autres trailers.

## Mettre à jour

Le dépôt et la configuration installée sont deux copies distinctes. Reporter les changements souhaités dans les fichiers versionnés ; ne pas recopier tout le dossier de configuration Zed dans Git.

Pour récupérer les réglages du dépôt, fermer Zed, puis :

```bash
cd "$HOME/personal_projects/zed-config"
git pull --ff-only
```

Sous PowerShell :

```powershell
Set-Location (Join-Path $HOME "personal_projects\zed-config")
git pull --ff-only
```

Relancer ensuite **l'étape de sauvegarde et de copie** correspondant au système. Elle conserve une sauvegarde locale et réapplique la conversion Cmd vers Ctrl sur Windows/Linux. Conserver `auto_install_extensions.jetbrains-icons` dans les réglages exportés.

Avant tout commit de configuration, examiner les fichiers et le diff pour rechercher des secrets ou des informations propres à un client. `.gitignore` limite les fichiers acceptés, mais ne détecte pas les secrets ajoutés à un fichier déjà suivi.

## Sécurité et limites

- Ne jamais versionner de mot de passe, clé API, jeton, clé SSH privée, fichier `.env` ou configuration confidentielle d'un client, même dans un dépôt privé.
- L'authentification GitHub est gérée séparément par `gh`, hors de ce dépôt. Ne pas copier son stockage d'identifiants, le trousseau système, les sessions Zed, les logs ou les sauvegardes dans le dépôt.
- Les copies de sauvegarde peuvent contenir des données sensibles. Elles restent sur la machine et ne doivent pas être publiées.
- Les paramètres IA exportés pointent uniquement vers le serveur Ollama local pour l'autocomplétion et les messages de commit. Garder cette adresse locale sauf accord explicite pour un autre service.
- Les commandes Windows et Linux doivent être vérifiées sur les machines cibles. Les restrictions de l'entreprise, les pilotes et les droits d'installation peuvent nécessiter une intervention de l'administrateur.

## Sources

- [Installation de Zed](https://zed.dev/docs/installation)
- [Zed sur Windows](https://zed.dev/docs/windows) et [sur Linux](https://zed.dev/docs/linux)
- [Ollama sur Windows](https://docs.ollama.com/windows) et [sur Linux](https://docs.ollama.com/linux)
- [Inter 4.1](https://github.com/rsms/inter/releases/tag/v4.1)
- [JetBrains Mono 2.304](https://github.com/JetBrains/JetBrainsMono/releases/tag/v2.304)
