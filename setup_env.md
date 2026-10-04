# LLMs-from-scratch on RunPod — Setup & Daily Routine

Repo: `https://github.com/Zohreh-ai-nik/LLMs-from-scratch` (fork of `rasbt/LLMs-from-scratch`)
Volume: `agentic_storage` (EU-RO-1) → mounted at `/workspace`
Project folder: `/workspace/LLMs-from-scratch`

> Everything outside `/workspace` is wiped when the pod stops. Everything inside `/workspace` persists.

---

## 1. Deploying a pod

1. RunPod → **Pods** → **Deploy**
2. Template: **Runpod PyTorch**
3. Region / volume: **`agentic_storage` (EU-RO-1)**
4. GPU: **RTX 2000 Ada** (or any cheap GPU with ≥16 GB VRAM)
5. **Deploy Pod** → wait for **Running**

If a stopped pod won't **Start** ("no GPU available"): **Terminate** it (safe — files are on the volume) and deploy a new one as above.

---

## 2. Connecting via SSH (laptop)

1. RunPod → pod → **Connect** → copy *SSH over exposed TCP* (IP + port)
2. Update the laptop SSH config:
   ```bash
   nano ~/.ssh/config
   ```
   ```
   Host runpod
       HostName <IP>
       Port <PORT>
       User root
       IdentityFile ~/.ssh/id_ed25519
   ```
3. If SSH warns **"REMOTE HOST IDENTIFICATION HAS CHANGED"**:
   ```bash
   ssh-keygen -R "[<IP>]:<PORT>"
   ```
4. VSCode → `Ctrl+Shift+P` → **Remote-SSH: Connect to Host** → `runpod`
5. **File → Open Folder** → `/workspace/LLMs-from-scratch`
6. Reinstall **Python** + **Jupyter** extensions if VSCode asks

---

## 3. Environment setup after connecting

### Step 1 — Load environment
```bash
source /workspace/start-llms.sh
```
Prompt should start with `(LLMs-from-scratch)`.

### Step 2 — Check
```bash
which python
python -c "import torch, tiktoken; print('OK', torch.__version__, 'CUDA:', torch.cuda.is_available())"
```
Expected:
- `/workspace/LLMs-from-scratch/.venv/bin/python`
- `OK 2.x.x CUDA: True`

✅ Works → skip to Step 4.

### Step 3 — Only if Step 2 fails: rebuild venv
```bash
cd /workspace/LLMs-from-scratch
deactivate 2>/dev/null
mv .venv /workspace/.old-venv-$(date +%s) 2>/dev/null
uv venv --python 3.10 && \
source .venv/bin/activate && \
uv pip install -r requirements.txt && \
uv pip install ipykernel nbstripout && \
nbstripout --install && \
python -c "import torch, tiktoken; print('OK', torch.__version__, 'CUDA:', torch.cuda.is_available())"
```
Then:
```bash
rm -rf /workspace/.old-venv-*
```

### Step 4 — Get latest code
```bash
git pull
```

### Step 5 — Open a notebook
Open a notebook → **Select Kernel** → **Python Environments** →
`/workspace/LLMs-from-scratch/.venv/bin/python`

---

## 4. Before leaving

```bash
cd /workspace/LLMs-from-scratch
git add . && git commit -m "progress" && git push
```
Then RunPod → **Stop** the pod.

| Action | Effect |
|---|---|
| **Stop** | GPU billing stops, pod + volume kept |
| **Terminate** | Pod deleted; `agentic_storage` volume still kept |

---

## 5. Occasional tasks

| When | Command |
|---|---|
| Author updated the book code | `git fetch upstream && git merge upstream/main && git push` |
| Install an extra package | `uv pip install <name>` (venv active) |
| Check volume usage | `du -sh /workspace` (compare with size in RunPod → Storage) |
| Find what's using space | `du -sh /workspace/* /workspace/.[!.]* 2>/dev/null \| sort -h` |

---

## 6. Troubleshooting

| Symptom | Fix |
|---|---|
| `Quota exceeded (os error 122)` | Volume full. Clear caches: `rm -rf /workspace/.cache/uv /workspace/.uv-cache/*`, or enlarge volume in RunPod → Storage |
| `Cache is currently in-use, waiting...` | A stuck uv process. `ps aux \| grep -i "[u]v "` → `kill -9 <PID>` |
| `rm: Directory not empty` on `.venv` | Use `mv .venv /workspace/.old-venv-$(date +%s)` instead |
| `ModuleNotFoundError` in notebook | Wrong kernel — select `/workspace/LLMs-from-scratch/.venv/bin/python` |
| `uv: command not found` | Run `source /workspace/start-llms.sh` |
| `git push` asks for password | Use a GitHub fine-grained token (Contents: Read & write) — saved after first use |

---

## 7. Reference: `/workspace/start-llms.sh`

```bash
#!/bin/bash
# uv: everything on the persistent volume, no cache
export UV_PYTHON_INSTALL_DIR=/workspace/.uv/python
export UV_PYTHON_PREFERENCE=only-managed
export UV_NO_CACHE=1
export UV_LINK_MODE=copy
export PATH="$HOME/.local/bin:$PATH"
command -v uv >/dev/null || curl -LsSf https://astral.sh/uv/install.sh | sh

# git identity + saved token
git config --global user.name "Zohreh"
git config --global user.email "<your-github-email>"
git config --global credential.helper "store --file=/workspace/.git-credentials"

# project + venv
cd /workspace/LLMs-from-scratch && source .venv/bin/activate
```




# make sure ipykernel is installed inside venv
source /workspace/start-llms.sh
python -c "import ipykernel; print('ipykernel OK')"

#if you get an error install it
uv pip install ipykernel

#Step 2: Make sure the extensions are on the pod

Open the Extensions panel (Ctrl+Shift+X). Python and Jupyter should show as installed in SSH: runpod. If they show an "Install in SSH: runpod" button, click it.

Step 3: Select the kernel
Open a notebook, for example ch02/01_main-chapter-code/ch02.ipynb.
Click Select Kernel in the top right.
Choose Python Environments…
Pick .venv (Python 3.10.x) /workspace/LLMs-from-scratch/.venv/bin/python.