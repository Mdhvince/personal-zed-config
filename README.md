# zed-config

**Ce Mac est la seule source de modifications. Les autres PC récupèrent la configuration, sans la modifier ni la pousser.**

Le dépôt est directement dans le dossier utilisé par Zed : aucune copie supplémentaire.

## Installer sur un autre PC

### 1. Installer les outils

macOS, avec [Homebrew](https://brew.sh/) :

```bash
brew install git gh
brew install --cask zed ollama-app font-inter font-jetbrains-mono
export PATH="/Applications/Ollama.app/Contents/Resources:$PATH"
```

Windows, avec WinGet dans PowerShell :

```powershell
winget install --exact --id Git.Git --source winget
winget install --exact --id GitHub.cli --source winget
winget install --exact --id ZedIndustries.Zed --source winget
winget install --exact --id Ollama.Ollama --source winget
```

Rouvrir PowerShell après l'installation.

Ubuntu / Debian :

```bash
sudo apt-get update
sudo apt-get install -y git gh curl ca-certificates libvulkan1 mesa-vulkan-drivers
curl -fsSL https://zed.dev/install.sh | sh
curl -fsSL https://ollama.com/install.sh | sh
```

Sur Windows/Linux, installer aussi **Inter Variable** et **JetBrains Mono** depuis les archives officielles : [Inter](https://github.com/rsms/inter/releases/download/v4.1/Inter-4.1.zip), [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono/releases/download/v2.304/JetBrainsMono-2.304.zip). Utiliser l'installateur de polices du système pour les fichiers `.ttf`.

### 2. Cloner dans le dossier Zed

Fermer Zed. Le dossier cible doit être absent ou vide. S'il contient une ancienne configuration, le sauvegarder et le renommer avant le clonage.

Se connecter au compte GitHub ayant accès au dépôt privé :

```bash
gh auth login --hostname github.com --git-protocol https --web
```

Exécuter uniquement la commande correspondant au système :

```bash
# macOS
gh repo clone Mdhvince/zed-config "$HOME/.config/zed"
```

```powershell
# Windows
gh repo clone Mdhvince/zed-config "$env:APPDATA\Zed"
```

```bash
# Linux
gh repo clone Mdhvince/zed-config "${XDG_CONFIG_HOME:-$HOME/.config}/zed"
```

### 3. Installer les modèles locaux

Démarrer Ollama (application, ou `ollama serve` dans un autre terminal), puis :

```bash
ollama pull qwen2.5-coder:7b-base
ollama pull qwen2.5-coder:7b-instruct
```

Les deux modèles occupent environ 9,4 Go. Ouvrir ensuite Zed ; l'extension JetBrains Icons s'installe automatiquement.

## Publier depuis ce Mac uniquement

Vérifier `git diff HEAD` avant publication : aucun mot de passe, jeton ou clé API ne doit être ajouté.

```bash
cd "$HOME/.config/zed"
git add -u
git commit -m "Update Zed configuration"
git push
```

## Mettre à jour les autres PC

Fermer Zed, ouvrir un terminal dans le dossier cloné, puis :

```bash
git pull --ff-only
```

Rouvrir Zed. Rien à recopier, aucun commit ni push à faire sur ces PC.

Sur un ordinateur client, respecter les autorisations d'installation. Les sauvegardes et autres fichiers locaux sont exclus par `.gitignore`, pas les secrets éventuellement ajoutés à un fichier suivi.
