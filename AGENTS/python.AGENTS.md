# Python — AGENTS Protocol

## Python

### Minimum Required Linters

- **ruff** — fast all-in-one linter and formatter (replaces flake8, isort, pyflakes, etc.)
- **mypy** — static type checker with `--strict` flag
- **pylint** — deep static analysis (yes, in addition to ruff — they catch different things)

All three must pass with **zero** warnings, **zero** errors, **zero** suppressions.

Install if missing:
```bash
pip install ruff mypy pylint
```

Run:
```bash
ruff check . && ruff format --check . && mypy --strict . && pylint **/*.py
```

### Type Hints Are Mandatory

Every function signature must have complete type annotations. No exceptions.
`Any` is not a type hint — it's a surrender. If you reach for `Any`, you
don't understand the data flow well enough yet.

```python
# GOOD
def parse_response(data: bytes, encoding: str = "utf-8") -> dict[str, list[int]]:
    ...

# BAD
def parse_response(data, encoding="utf-8"):
    ...

# TERRIBLE
def parse_response(data: Any) -> Any:
    ...
```

### No Bare Excepts, No Broad Excepts

```python
# GOOD: Catch specific exceptions, handle them meaningfully
try:
    config = json.loads(raw_data)
except json.JSONDecodeError as exc:
    raise ConfigError(
        f"Failed to parse config from {source}: {exc}"
    ) from exc

# BAD: Catches everything including KeyboardInterrupt and SystemExit
try:
    config = json.loads(raw_data)
except Exception:
    config = {}  # "sensible default" that masks the real problem

# FORBIDDEN: The silent killer
try:
    config = json.loads(raw_data)
except:
    pass
```

### String Formatting

Use f-strings. Not `%` formatting. Not `.format()`. f-strings are faster,
more readable, and harder to get wrong.

```python
# GOOD
msg = f"Connection to {host}:{port} failed after {retries} retries"

# BAD
msg = "Connection to %s:%d failed after %d retries" % (host, port, retries)

# ALSO BAD
msg = "Connection to {}:{} failed after {} retries".format(host, port, retries)
```

### Imports

Imports are at the top of the file. Always. No inline imports unless there
is a genuine circular dependency issue, and even then, restructure the code
to avoid it if at all possible.

```python
# GOOD
from pathlib import Path
from collections.abc import Sequence

def process(paths: Sequence[Path]) -> None:
    ...

# BAD
def process(paths):
    from pathlib import Path  # why is this here?
    ...
```

### Pathlib Over os.path

```python
# GOOD
from pathlib import Path

config_dir = Path.home() / ".config" / "myapp"
config_dir.mkdir(parents=True, exist_ok=True)
config_file = config_dir / "settings.json"
text = config_file.read_text(encoding="utf-8")

# BAD
import os

config_dir = os.path.join(os.path.expanduser("~"), ".config", "myapp")
os.makedirs(config_dir, exist_ok=True)
config_file = os.path.join(config_dir, "settings.json")
with open(config_file, "r") as f:
    text = f.read()
```

### Context Managers for Resources

If it opens, it closes. Period.

```python
# GOOD
with open(path, "r", encoding="utf-8") as fh:
    data = json.load(fh)

# BAD: file handle leak if json.load raises
fh = open(path, "r")
data = json.load(fh)
fh.close()
```

### Never Use Mutable Default Arguments

```python
# GOOD
def append_item(item: str, target: list[str] | None = None) -> list[str]:
    if target is None:
        target = []
    target.append(item)
    return target

# BAD: The list is shared across ALL calls
def append_item(item: str, target: list[str] = []) -> list[str]:
    target.append(item)
    return target
```

This is not a style preference. The "bad" version is a bug. The default
list is created once at function definition time and shared across all
calls. This is one of the most common Python traps and agents fall into it
regularly.

---

## Python — PyTorch and GPU Memory

PyTorch memory management is a minefield. Agents routinely produce code
that "works" on small inputs and OOMs on real data, or that slowly leaks
GPU memory until the process is killed.

### Minimum Required Linters (in addition to Python above)

All Python linters above, plus:
- **torchfix** — PyTorch-specific linter (`pip install torchfix`)

### Tensor Device Awareness

Every tensor operation must be explicit about device placement. Never assume
tensors are on a specific device.

```python
# GOOD
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
tensor = torch.zeros(batch_size, features, device=device)

# BAD: Creates on CPU, then must be moved (wasteful allocation)
tensor = torch.zeros(batch_size, features).to(device)

# WORSE: Assumes CUDA is available
tensor = torch.zeros(batch_size, features).cuda()
```

### Inference Must Use no_grad

Every inference code path must be wrapped in `torch.no_grad()`. Without it,
PyTorch builds a computation graph that will never be used for backprop,
wasting enormous amounts of memory.

```python
# GOOD
@torch.no_grad()
def predict(model: nn.Module, inputs: torch.Tensor) -> torch.Tensor:
    model.eval()
    return model(inputs)

# BAD: Builds computation graph for nothing, leaks memory
def predict(model, inputs):
    return model(inputs)
```

### Free GPU Memory Explicitly

```python
# GOOD: Release intermediate tensors
def train_step(model: nn.Module, batch: dict[str, torch.Tensor]) -> float:
    outputs = model(batch["input"])
    loss = criterion(outputs, batch["target"])
    loss.backward()
    loss_value = loss.item()  # extract scalar BEFORE deleting
    del outputs, loss
    torch.cuda.empty_cache()  # only when switching between large allocations
    return loss_value

# BAD: Intermediate tensors accumulate
def train_step(model, batch):
    outputs = model(batch["input"])
    loss = criterion(outputs, batch["target"])
    loss.backward()
    return loss.item()  # outputs tensor still referenced, holding memory
```

### DataLoader Workers and Pin Memory

```python
# GOOD: Efficient data loading
loader = DataLoader(
    dataset,
    batch_size=32,
    num_workers=4,      # parallel data loading
    pin_memory=True,    # faster CPU-to-GPU transfer
    persistent_workers=True,  # don't recreate workers each epoch
)

# BAD: Single-threaded data loading bottleneck
loader = DataLoader(dataset, batch_size=32)
```

### Gradient Accumulation Memory

```python
# GOOD: Proper gradient accumulation without OOM
optimizer.zero_grad()
for i, micro_batch in enumerate(micro_batches):
    loss = model(micro_batch) / len(micro_batches)
    loss.backward()  # gradients accumulate in .grad
    del loss  # free the computation graph
optimizer.step()

# BAD: Stores all computation graphs simultaneously
losses = [model(mb) for mb in micro_batches]
total_loss = sum(losses) / len(losses)
total_loss.backward()  # OOM: all graphs in memory at once
```

### Avoid .detach() When .item() Suffices

```python
# GOOD: For scalar metrics
accuracy = (preds == labels).float().mean().item()

# BAD: Creates a new tensor that may hold graph references
accuracy = (preds == labels).float().mean().detach().cpu().numpy()
```

### Mixed Precision Training

```python
# GOOD: Proper AMP usage
scaler = torch.amp.GradScaler()
for batch in loader:
    optimizer.zero_grad()
    with torch.amp.autocast(device_type="cuda"):
        output = model(batch["input"])
        loss = criterion(output, batch["target"])
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
    del output, loss

# BAD: Manual half-precision that breaks gradient scaling
model.half()  # breaks batch norm, loses precision everywhere
```

### Memory Leak Detection Pattern

```python
# Put this in your training loop during development
if step % 100 == 0:
    allocated = torch.cuda.memory_allocated() / 1024**2
    reserved = torch.cuda.memory_reserved() / 1024**2
    logger.debug(
        f"Step {step}: GPU allocated={allocated:.0f}MB, "
        f"reserved={reserved:.0f}MB"
    )
    # If 'allocated' grows monotonically, you have a leak
```

---

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
