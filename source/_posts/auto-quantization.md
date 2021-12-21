---
title: Auto Quantization in Physical Simulations
date: 2021-12-17 16:32:17
tags: taichi
thumbnail: 
# - /img/auto_quant/4quant.jpg
- /img/auto_quant/fluid.jpg
mathjax: true
---

In this paper, we propose a novel workflow to allow users to seamlessly tune their simulations for a smaller memory footprint, which is especially valuable for scaling the simulation to a higher resolution. Given an objective for compression rate or relative error bound, the whole process is automated as part of the compiler support, and each identified variable is statically assigned to an appropriate length-optimized type. Referencing the derivatives supplied by auto-diff support, we proposed a precise estimation of the round-off errors with respect to quantization schemes, allowing us to achieve much higher efficiency than Random Search in determining the quantization schemes. We implemented several examples to demonstrate the generality and efficacy of our method, in which the average compression rate is 1.9, without degradation of perceived physical feasibility in visual effects.  


(Coming soon, manuscript in preparation for *SIGGRAPH 2022*)

<!-- more -->

> **This version is for preview only. Usage without permission or proper reference is not allowed. **
@article{
  author    = {Liu, Jiafeng^* and Shi, Haoyang^* and Xu, Weiwei and Yang, Yin and Ma, Chongyang},
  title     = {Auto Quantization in Physical Simulations},
  year      = {2021},
  month     = dec,
  keywords  = {quantization, physical simulation, domain-specific language, compiler}
  author+an = {1=highlight; 2=highlight}
}


### Introduction
Previously, QuanTaichi ^[hu2021quantaichi] implemented a compiler-supported "quantized" type system, providing user control over representations of physical variables in memory to achieve a higher information density. 


Undoubtedly, the quantized type is a desirable invention for large-scale simulations with high resolutions. For the simulations on a single GPU, the introduction of lower-precision("quantized") types breaks the hard limit on the simulated particle count posed by the amount of available device-side memory by as much as 1.7 times. On the other hand, multiple-GPU simulations are benefited both on scale and speed, as the quantized types save a similar proportion on the bandwidth consumption. 

Reportedly, researchers are beginning to tap into the potential of quantizing in the field of artificial intelligence, where exhilarating results manifest that a quantized 8-bit neural network is x-times faster than its floating-point counterpart in training, yet still achieving the same level of accuracy ^[gholami2021survey].

However, we are not safe to assume the quantization as a 'harmless' gift until we find the answer to a series of questions on the influence of quantization on the simulated process. How much further can we compress the variables before the quality loss become intolerable, and how many bits of information are essential to encode the whole process? 
With the current *quantaichi* implementation, users have to tackle these core issues themselves, most likely by trial-and-error and completely void of systematic assistance.
 
The problem is weaved with the following troublesome characteristic: 

1. Quantization error tends to accumulate over time, eventually hijacking the ensuing simulation. Nevertheless, there are a series of simulations that are sensitive to the initial condition. As a result, point-to-point correspondence in the final result is almost impossible, giving rise to the necessity of an alternative definition of the loss of visual effect.

2. As the number of customized types grows, the search space fluctuates exponentially. This curse of dimension makes fine-grained customization almost impossible.

3. Some variables are obscure in their physical meanings, posing special hardness for non-expert users. e.g., the affine velocity field in APIC.

4. The sparsity of the controlled variable in the decision of fraction bits. It means that we must either solve a discrete optimization or apply relaxation to the original problem.

Inspired by the Error Propagation Formula, we consult the derivatives to estimate the impact of quantization error. We generalized the workflow to a compiler-supported automated quantizing support.
With our system, users can effortlessly achieve a human-compatible compression rate with a physically plausible visual effect. 

To achieve those benefits, users must specify a key measurement $z$ to indicate simulation quality, e.g. the energy function for energetic liquid simulation. Depending on their emphasis, users can choose their focus between fidelity and compression rate, by either appointing a relative error constrain or setting a target compression rate. 

Due to the precise estimation of quantization error, our method has the following benefits to distinguish itself from existing quantizing frameworks:

**Quantitative:** Depending on users' emphasis, we provide quantitative guarantees on the quality of the resultant quantizing scheme. If fidelity is the focus, our typical solution achieves a distribution of an average bias and standard variance of 0.7 and 1.1 times of error bound respectively. Or, for the user-specified compression rate, we guarantee to meet the constrain with minor exceptions due to up-rounding of the required fraction bits.

**Time-efficient:** We are able to arrive at a fair combination after a single run of the simulation, whereas the final solution lies only one or two bits away. For the specified error bound to hold, the median of iteration times for simulation is 2.

Our system is implemented as an extension to the Taichi programming language to take advantage of its auto-diff feature. However, our methods are general for simulations implemented on any differentiable programming system.

### Methods

#### Error Analysis
In the scope of this chapter, we highlight the quantization error introduced by each store to the global memory and its influence on the visual quality. 

For a given simulation, we formalize it as iteratively applying a function $\mathbf{X_t} = F(X_{t-1})$, which maps the current state to the next discrete time step, with $\mathbf{X_i}$ denoting the concatenation of physical variables bearing the information of time step i. With an infinite number of digits, the true values  $\mathbf{X_i^*}$ will be as the recursive definition:
$$
\begin{align}
    \mathbf{X_i^*} = F(\mathbf{X_{i-1}^*})
\end{align}  
$$
Additionally, we define the quantization step be H. In actual computation, the quantized values will be:
$$
\begin{align}
     \mathbf{X_i} = H(F(\mathbf{X_{i-1}}))
\end{align}
$$
Let us denote the target function supplied by the user as $z(\mathbf{X_t})$. To get high visual fidelity, we want the simulated z value to adhere to the true value $z(\mathbf{X_t^*})$ as closely as possible. 

To bridge the difference between $\mathbf{X_i^*}$ and $\mathbf{X_i}$, we introduce an intermediate 
$$
\begin{align}
    \widetilde{\mathbf{X_i}} \triangleq F(\mathbf{X_{i-1}})
\end{align}
$$
then the quantization error introduced in time step i is defined as:
$$
 \begin{align}
    \mathbf{\epsilon_i} \triangleq \mathbf{X_i}-\widetilde{\mathbf{X_i}}
 \end{align}
$$
If we have $b$ fraction bits for every variable, the quantization error is limited to:
$$ | \epsilon_{i_j} | < 2^{-(b+1) }, \forall j$$
We define 
$\mathbf{\delta_i} \triangleq \mathbf{X_i} - \mathbf{X_i^*}$
as the difference between the true value and the quantized value.
$$
\begin{equation}
\begin{split}
\mathbf{\delta_i} &= (\mathbf{X_i}-\widetilde{\mathbf{X_i}}) +(\widetilde{\mathbf{X_i}} - \mathbf{X_i^*}) \\
                &= \mathbf{\epsilon_i} + (F(\mathbf{X_{i-1}})-F(\mathbf{X_{i-1}^*)})
    \end{split}
\end{equation}
$$
Expand $F(\mathbf{X})$ at point $\mathbf{X_{i-1}^*}$
$$
\begin{equation}
    \begin{split}
        \mathbf{\delta_i} &= \mathbf{\epsilon_i} + J_{i,i-1}(\mathbf{X^*_{i-1}}-\mathbf{X_{i-1})}+ o(\delta_{i-1}) \\
        &= \mathbf{\epsilon_i} + J_{i,i-1}\delta_{i-1}+ o(\delta_{i-1}) \\
        &\approx \mathbf{\epsilon_i} + \sum_{j=0}^{i-1} {\prod_{k=j}^{i-1}{J_{k,k-1}\mathbf{\epsilon_j}}} \\
        &= \sum_{j} { J_{i,j}\mathbf{\epsilon_j}}
    \end{split}
\end{equation}
$$
where $J_{i,j} = \frac{\partial{\mathbf{X_i}}}{\partial{\mathbf{X_j}}} |_{\mathbf{X_j}=\mathbf{X_j^*}}$ is the Jacobian matrix.

We then expand $z$ at $\mathbf{X_t^*}$

$$
\begin{equation}
    \begin{split}
    z(\mathbf{X_t}) &= z(\mathbf{X_t^*}) + \frac{\partial{z}}{\partial{\mathbf{X_t}}}|_{\mathbf{X_t}} \delta_t +     o(\delta_t) \\
    &\approx z(\mathbf{X_t^*}) + \frac{\partial{z}}{\partial{\mathbf{X_t}}}|_{\mathbf{X_t^*}}\sum_i J_{t,i} \mathbf{\epsilon_i} \\
    &= z(\mathbf{X_t^*}) + \sum_i f_i\mathbf{\epsilon_i}
    \end{split}
\end{equation}
$$

where $f_i$ is the notation for $\frac{\partial{z}}{\partial{\mathbf{X_i}}}|_{\mathbf{X_i} = \mathbf{X_i^*}}$. The variance of z is

$$
\begin{align}
    Var(z(\mathbf{X_t}))=\sum_{i,j}f_i f_jCov(\mathbf{\epsilon_i}, \mathbf{\epsilon_j})
\end{align} 
$$

With independence assertion, the estimation of the variance of $z$ is simplified to 
$$
\begin{align}
    \sigma_z  = \sqrt{\sum_{t} {\sum_i f_{x_{t,i}^2\sigma_{i}^2}}}
\end{align}
$$

If enforced by the user, a relative error constrain holds as follow:
$$
\begin{align}
    \frac{\sigma_z}{z} < \epsilon
\end{align} 
$$

For fixed-point representation, assuming uniform distribution, the relationship between the number of fraction bits and the standard variance is 

$$
\begin{align}
    \sigma_{x_{t,i}} = 2^{-(b_i+1)}/\sqrt3 
\end{align} 
$$

For the floating-point representation, the relative error is constrained by the number of fraction bits as

$$
\begin{align}
    \frac{\sigma_{x_{t,i}}}{x_{t,i}} < 2^{-(b_i+1)}
\end{align}
$$

Now, assuming fixed-point representation, We have transformed the problem into an optimization:

$$
\begin{align}
    \min_{\sigma_i}    \ \sum_i b_i \\
    s.t.    \ \sum_t \sum_{i}f_{x_{t,i}}^2\sigma_i^2 \le C \\
    where   \ b_i = -log_2(2\sqrt3 \sigma_i)
\end{align}
$$

Conversely, if a target compression rate $\lambda$ is offered, the optimization changes to:

$$
\begin{align}
 \min_{\sigma_i}    \ \sum_t \sum_{i}f_{x_{t,i}}^2\sigma_i^2 \\
 s.t.    \frac {1}{n} \sum_i^n b_i< 32 \lambda \\
\end{align}
$$

where   $\ b_i = -log_2(2\sqrt3 \sigma_i)$

In optimization (12) and (13), all the coefficients $f_{x_{t,i}}$ can be computed with taichi's auto differential feature. 


In a sense, the dependence on gradient computation will impose a stronger limit on the application of our system, with increased complexity in time and space and the requirement of a differentiable implementation. However, this is not an unavoidable loss of generality. For the extra constraints on differentiable programming, users can refer to *DiffTaichi*^[DiffTaichi] and modify the memory access part of their programs accordingly. Additionally, we extended the checkpointing method in *DiffTaichi* to further alleviate the space complexity, making the differentiable framework more practical. 

So far, we have been avoiding the discussion on the legality of formulae (8). In the following chapters, we will take a hard look at how it might affect the constraints if its underlying premises deteriorates.


#### 2.2 Error Control

Previously, we modeled the quantization error as random uniform distribution for each quantization operation and asserted independence between each operation. However, it can be too optimistic an estimation. The quantization error is directly linked with the digit after the least significant bit, which, arbitrary as it might seem, is liable to reflect the functional relationship during the iteration between timesteps or the interaction of adjacent particles. In other words, if the independence assertion on the truncated digits $Cov(d_i^t, d_i^{t+1})=0 $ or $Cov(d_j^t, d_i^t)=0$ does not hold, it is not safe to apply the formula, resulting in a much-escalated estimation of the error bound. 

As tracing the covariance would be computationally infeasible, we take a detour by dealing with the correlations on the truncated digits.

By imposing a random noise on the digit in question, we decoupled the correlation between the original data and the quantization error, achieving an accurate estimation of the error. To elaborate, in each store operations, we impose an unbiased additive uniform noise with magnitude $[-2^{-(lsb+1)}, 2^{-(lsb+1)}]$ before rounding.

The benefit of dithering comes in threefold:
1. Dithering supplies the required conditions for the round-off error to be uniform and white^[Noise]. By our experiments, independence assertion should not be taken for granted.

2. The dithered data is a more accurate description of the original number. Even when correlations do occur, our dithered data processes a stronger tolerance, managing a less biased result. To understand that, if we denote the truncated digits (the digits after the LSB) as $y \in [0,1)$ and the random noise as $\xi \in [-0.5, 0.5]$, the dithered data after rounding $q(y + \xi)$ shares the same expectation as y. In comparison, if y distributes unevenly between the two intervals $[0, 0.5),[0.5,1)$ in the entire simulation, the naive rounding scheme will result in an insufferable bias.

3. The randomness introduced by dithering does not harm the number of valid digits, as it only tampers the least significant digit by at most 1. 



### Preliminary Results

**Video:** Comprarison of float 64 ground truth and resultant simulation with our derived quantization scheme. The compression rate is as high as 2.5x but no distinguishable artifacts could be recongnized. 
{% raw %}
<video src="/videos/compression.mp4" type='video/mp4' controls='controls' width='100%' height='100%'>
</video>
{% endraw%} 
**Top left:** float64. **Top right:** compression rate 1.7. **Bottom left:** compression rate 2.0. **Bottom right:** compression rate 2.5. 
