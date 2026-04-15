# Conda Environment Build Workflow

This workflow creates self-contained conda environments with the `claw` Rust binary pre-installed, allowing users to download and run without any additional installation.

## How It Works

1. **Creates a conda environment** with Python 3.11, Rust 1.75, and Cargo from conda-forge
2. **Builds the Rust project** using `cargo build --release -p rusty-claude-cli`
3. **Installs the binary** into the conda environment's `bin/` directory
4. **Packages the entire environment** as a 7z archive (split into 800MB chunks)
5. **Uploads to GitHub Release** (optional) or as workflow artifacts

## Usage

### Trigger the Workflow

Go to **Actions** > **Build Conda Environment** > **Run workflow**

### Inputs

| Input | Description | Default | Options |
|-------|-------------|---------|----------|
| `platform` | Target OS | `windows` | `windows`, `linux`, `macos` |
| `rust_toolchain` | Rust version | `stable` | Any rust-toolchain version |
| `build_profile` | Cargo profile | `release` | `release`, `debug` |
| `retention_days` | Artifact retention | `30` | Integer |
| `create_release` | Create GitHub release | `false` | `true`/`false` |

### Example: Build Windows Environment

1. Go to Actions tab
2. Select "Build Conda Environment" workflow
3. Set `platform` to `windows`
4. Click "Run workflow"
5. Wait for completion (usually 5-10 minutes)
6. Download the `.7z` artifacts from the workflow run

## Downloading and Using

### Step 1: Download

Download all parts of the archive (e.g., `claw-env-windows-v0.1.0.7z.001`, `.7z.002`, etc.)

### Step 2: Extract

Place all parts in the same directory and extract using 7-Zip:

```bash
# Windows (PowerShell)
7z x claw-env-windows-v0.1.0.7z.001 -oD:\conda_envs

# Linux/Mac
7z x claw-env-windows-v0.1.0.7z.001 -o/home/user/conda_envs
```

### Step 3: Activate

```bash
# Windows
call "D:\conda_envs\claw-env\Scripts\activate.bat"

# Linux/Mac
source /home/user/conda_envs/claw-env/bin/activate
```

### Step 4: Run

```bash
claw --help
```

## Local Testing

To test locally without GitHub Actions:

```bash
# Create environment
conda env create -f .github/workflows/environment.yml

# Activate
conda activate claw-env

# Build and install manually
cd rust
cargo build --release -p rusty-claude-cli
cp target/release/claw.exe ../../claw-env/bin/  # Windows
cp target/release/claw ../../claw-env/bin/       # Linux/Mac

# Test
claw --help
```

## Architecture

The workflow follows this pattern:

```
Checkout → Setup Miniforge → Setup Rust → Create Conda Env → 
Build Cargo → Copy Binary to Env/bin → Verify → 
Export environment.yml → Compress with 7z → Upload
```

## File Structure After Extraction

```
claw-env/
├── bin/
│   ├── claw (or claw.exe)
│   ├── python
│   ├── conda
│   └── ...
├── Lib/ (or lib/)
├── Scripts/ (Windows only)
└── ...
```

## Notes

- The environment is **self-contained** but requires 7-Zip to extract
- Archives are split into 800MB chunks for GitHub's file size limits
- The Rust binary is built with the `release` profile by default for optimal performance
- Conda environment uses `conda-forge` channel for maximum compatibility
