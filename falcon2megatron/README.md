# Falcon and Megatron

This file documents the usage of all currently-implemented falcon features in the megatron project.
The main features are:
1. **Falcon weight converting**.
   Converting existing falcon weights to a megatron-compatible format.
1. **Falcon correctness verification**.
   Verifying that the current implementation of falcon in the megatron code matches the official implementation.
1. **Falcon checkpoint splitter**.
   Split the checkpoint saved in one file into multiple files, in order to use model parallelism.
1. **Falcon tokenizing**.
1. **Falcon training**.
   Train falcon :D

## Falcon weight converting

The main file to do the weight conversion is `falcon2megatron/falcon2megatron.py`.
This file fetches the weights directly from the [huggingface implementation](https://huggingface.co/tiiuae/falcon-40b), so no additional files are required to run the script.
To extract, the 40B model for instance, run:
```
python falcon2megatron.py --size=40 --out=/path/to/output/directory/
```
This uses huggingface default cache directory to store the original weights.
To change the cache use:
```
python falcon2megatron.py --size=40 --out=/path/to/output/directory/ --cache-dir=/path/to/huggingface/cache/directory/
```

See also `examples/fetch_falcon.sh`.

## Falcon correctness verification

**Warning**: The current code does not support model-parallelism, this is still work in progress.

To verify that the current implementation of falcon in the megatron code is correct, use the file `verify_correctness.py`.
See for instance `examples/verify_falcon.sh`.
Make sure to set the `--model_size=7 or `40` (depending on whether you test the 7B or 40B) and to use the /path/to/output/directory/ selected in the previous step as the `--load` argument.
Example outputs at this stage are:
```
...
Iteration 0...
Max absoulute error in the logits: 0.029 Abs loss error: 0.002
Iteration 1...
Max absoulute error in the logits: 0.037 Abs loss error: 0.002
Iteration 2...
Max absoulute error in the logits: 0.067 Abs loss error: 0.001
Iteration 3...
Max absoulute error in the logits: 0.051 Abs loss error: 0.001
...
```

Also, make sure to remove the `--bfp16` flag to use the 32-bit model and get higher precision outputs.

## Falcon checkpoint splitter

In order to use model parallelism you need to split the previously converted weights into multiple files.
To do this, use `tools/checkpoint_util.py`.
See for instance `examples/parallelize_falcon.sh`.
Once the weights are splitted, make sure to use the new checkpoint directory (set in the `--save_dir` argument) and the same tensor/model paralellism levels when running the next step.

**Note**: If you are using a docker image to run this step and get an error similar to `17300 Bus error (core dumped)` during this step, try increasing the shared memory size of the container (set to `--shm-size=128gb` in my experiments for falcon 40B).

## Falcon tokenizing

Work in progress...

## Falcon training

Use the `finetune_falcon.py`.
See for instance `examples/finetune_falcon.sh`.
Don't forget to set the tensor and pipeline paralellism levels to the numbers set in the previous step.
The loss should decrease across iterations.

**Important**:
- No data preprocessing has been done using falcon tokenizer yet, so the initial loss is still very high.
- Falcon-40B does not fit in 8 GPUs when trying to train, so it hasn't been tested yet.