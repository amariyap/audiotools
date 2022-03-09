# Transforms

<!-- ## Running this notebook

This notebook creates a model card for a specified model checkpoint. To run
this notebook, you must ensure that `pandoc` and `codebraid` are installed:

```
# https://pandoc.org/installing.html#linux
pip install codebraid
```

The notebook can be run and its output can be copy/pasted to Discourse via:

```
python -m audiotools.post --discourse notebooks/transforms.md > notebooks/transforms.exec.md
```

The contents of `fuzziness.exec.md` can then be copy-pasted to Discourse.
You can also view the contents without uploading to Discourse by outputting to HTML:

```
python -m audiotools.post notebooks/transforms.md > notebooks/transforms.html
```

Which you can then open in a browser to view. -->

```{.python .cb.nb show=code:none+rich_output+stdout:raw+stderr jupyter_kernel=python3}
from audiotools import AudioSignal
from audiotools import post, util
import audiotools.data.transforms as tfm
from audiotools.data import preprocess

audio_path = "tests/audio/spk/f10_script4_produced.wav"
signal = AudioSignal(audio_path, offset=10, duration=2)

preprocess.create_csv(
    util.find_audio("tests/audio/nz"),
    "/tmp/noises.csv"
)
preprocess.create_csv(
    util.find_audio("tests/audio/ir"),
    "/tmp/irs.csv"
)

transform = tfm.Compose([
    tfm.Silence(prob=0.1),
    tfm.LowPass(),
    tfm.RoomImpulseResponse(csv_files=["/tmp/irs.csv"]),
    tfm.BackgroundNoise(csv_files=["/tmp/noises.csv"]),
    tfm.ClippingDistortion(),
    tfm.MuLawQuantization(),
    tfm.VolumeChange(),
])

outputs = {}

for seed in range(10):
    output = {}

    batch = transform.instantiate(seed, signal)
    batch["signal"] = signal.clone()
    batch = transform(batch)

    output["input"] = batch["original"]
    params = batch["Compose"]
    for k1, v1 in params.items():
        if isinstance(v1, dict):
            for k2, v2 in v1.items():
                output[f"{k1}.{k2}"] = v2
    output["output"] = batch["signal"]
    outputs[seed] = output

post.disp(outputs, first_column="seed")
```