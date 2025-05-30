# **Quarto-Jupyter Multi-Language Configuration**
Cross-language execution requires special handling:
- Default limitation: One kernel per notebook session
- Solution: Use polyglot kernels like SoS/Polyglot

Must declare handler in YAML:

```md
---
title: "Multi-Lang Doc"
jupyter: sos-jupyter
---
```

Kernel manages Python/R/Julia through cell magics

# **Julia Multi-Language Configuration**
Cross-language execution solutions for seamless integration:
- Native interoperability: Julia provides direct language bridges
- PythonCall.jl: Direct Python interoperability within Julia sessions
- RCall.jl: Embedded R code execution from Julia environment

Activation: 
```julia

using PythonCall 
py"..."

using RCall
R"..."
```