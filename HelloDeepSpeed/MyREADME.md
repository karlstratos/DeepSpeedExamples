# Setup

```
conda create --name deepspeed-examples python=3.8
conda activate deepspeed-examples
conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
pip install -r requirements.txt
pip install tensorboard
pip install deepspeed
```

## Trouble I

**Error message**: `packaging.version.InvalidVersion: Invalid version: '0.10.1,<0.11'`

**Issue**: `packaging` (which is not controlled in requirements, ugh) incompatibility, ["format requirements in new packaging version checking"](https://github.com/CompVis/latent-diffusion/issues/207)

**Solution**: Downgrade `packaging`
```
pip install packaging==21.3
```

Note that this trouble has nothing to do with deepspeed, it's just a stupid third-party library issue.


## Trouble II

**Error message**:
```
  c++ -MMD ... -std=c++17
  .../scaled_upper_triang_masked_softmax.cpp:17:10: fatal error: cuda_fp16.h: No such file or directory
     17 | #include <cuda_fp16.h>
        |          ^~~~~~~~~~~~~
  compilation terminated.
```

**Issue**: The compiler is not able to find the header, which exists in my CUDA place as Tim demonstrates:
```
find /usr/local/cuda-11* -name 'cuda_fp16.h'
/usr/local/cuda-11.7/targets/x86_64-linux/include/cuda_fp16.h
/usr/local/cuda-11.8/targets/x86_64-linux/include/cuda_fp16.h
```

**Solution**: Tell the compiler where to find it. Typically this is done by adding `-I` flag to the compilation command. Since I'm not directly running the command, I can set the environment variable [`CPLUS_INCLUDE_PATH`](https://gcc.gnu.org/onlinedocs/cpp/Environment-Variables.html) as follows:
```
export CUDAPATH=/usr/local/cuda  # Choosing the system-wide cuda, which is 11.7 according to /usr/local/cuda/bin/nvcc --version
export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:${CUDAPATH}/targets/x86_64-linux/include
```
Side note: I didn't export `CUDAPATH` but it worked, which is weird according to [this post](https://askubuntu.com/questions/205688/whats-the-difference-between-set-export-and-env-and-when-should-i-use-each).
```
(deepspeed-examples) jl2529@one:~/repositories/DeepSpeedExamples/HelloDeepSpeed$ echo ${CUDAPATH}

(deepspeed-examples) jl2529@one:~/repositories/DeepSpeedExamples/HelloDeepSpeed$ CUDAPATH=/usr/local/cuda
(deepspeed-examples) jl2529@one:~/repositories/DeepSpeedExamples/HelloDeepSpeed$ echo ${CUDAPATH}
/usr/local/cuda
``

### Apex

More generally, the trouble is that deepspeed is incapable of installing ops as shown [here](https://www.deepspeed.ai/tutorials/advanced-install/) because it can't find header files.

Apex is needed for things like FusedAdam. After fixing the path problem, we can install apex by
```
git clone https://github.com/NVIDIA/apex
cd apex
pip install -v --disable-pip-version-check --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./
```
