# personal-zed-config

## macOS

```bash
brew install git
brew install --cask zed ollama-app font-inter font-jetbrains-mono
export PATH="/Applications/Ollama.app/Contents/Resources:$PATH"
```

## Windows, avec WinGet dans PowerShell :

```powershell
winget install --exact --id Git.Git --source winget
winget install --exact --id ZedIndustries.Zed --source winget
winget install --exact --id Ollama.Ollama --source winget
```

Rouvrir PowerShell après l'installation.

## Ubuntu / Debian :

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates libvulkan1 mesa-vulkan-drivers
curl -fsSL https://zed.dev/install.sh | sh
curl -fsSL https://ollama.com/install.sh | sh
```

Sur Windows/Linux, installer aussi **Inter Variable** et **JetBrains Mono** depuis les archives officielles : [Inter](https://github.com/rsms/inter/releases/download/v4.1/Inter-4.1.zip), [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono/releases/download/v2.304/JetBrainsMono-2.304.zip). Utiliser l'installateur de polices du système pour les fichiers `.ttf`.

### 2. Cloner dans le dossier Zed

Fermer Zed. Le dossier cible doit être absent ou vide. S'il contient une ancienne configuration, le sauvegarder et le renommer avant le clonage.

Le dépôt est public : aucune connexion GitHub n'est nécessaire.

Exécuter uniquement la commande correspondant au système :

```bash
# macOS
git clone https://github.com/Mdhvince/personal-zed-config.git "$HOME/.config/zed"
```

```powershell
# Windows
git clone https://github.com/Mdhvince/personal-zed-config.git "$env:APPDATA\Zed"
```

```bash
# Linux
git clone https://github.com/Mdhvince/personal-zed-config.git "${XDG_CONFIG_HOME:-$HOME/.config}/zed"
```

### 3. Installer les modèles locaux

Démarrer Ollama (application, ou `ollama serve` dans un autre terminal), puis :

```bash
ollama pull qwen2.5-coder:7b-base
ollama pull qwen2.5-coder:7b-instruct
```

Les deux modèles occupent environ 9,4 Go. Ouvrir ensuite Zed ; l'extension JetBrains Icons s'installe automatiquement.
