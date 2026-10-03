# Developer Cookbook — PAX_QUANTIZER
**Stack:** Python 3.11, llama.cpp, bitsandbytes, auto-gptq, AIOSS_FORMAT

## Basic Usage
```python
from pax_quantizer import Quantizer
module = Quantizer(pax_model="./pax-27b-q4.gguf",
                               aioss_chain="./pax_quantizer.aioss")
result = module.process(input_data)
print(result.output, result.chain_hash)
```

## Batch Processing
```python
results = module.process_batch(inputs, batch_size=4)
for r in results:
    print(r.chain_hash)
```

## AIOSS Append
```python
import hashlib, time
def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

chain_hash = aioss_append("./pax_quantizer.aioss", result.to_bytes(), "PAX_QUANTIZER")
```

## Integration with Anticloud TIER_2
```python
# Chain with PAX_INFERENCE_CORE
from pax_inference_core import PAXInferenceCore
from pax_quantizer import Quantizer

core = PAXInferenceCore(model="./pax-27b-q4.gguf")
module = Quantizer(inference_core=core)
```

## Domain: Weight quantization pipeline: GGUF/GPTQ/AWQ for PAX 27B
This module specializes in: weight quantization pipeline: gguf/gptq/awq for pax 27b.
AIOSS entry type: quantized artifact (original weight hash + quantized weight hash + quality metrics).

## Performance
Use module.benchmark() to measure throughput on your hardware.
Pre-warm: module.warmup() before serving production requests.
