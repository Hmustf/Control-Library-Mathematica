# My Control Library for Mathematica

## ****Function Description****

### **`PlotStepResponse`**

The **`PlotStepResponse`** function computes and plots the step response of a given transfer function model. It automatically detects the type of the system (continuous or discrete) based on the transfer function and generates the appropriate response plot.

### Syntax

```mathematica
PlotStepResponse[transferFunctionModel, timeDuration]
```

- **`transferFunctionModel`** - A transfer function model created using Mathematica's **`TransferFunctionModel`**.
- **`timeDuration`** - The time duration for which the step response is to be plotted.

### Implementation

```mathematica
PlotStepResponse[TFM_, duration_ : Automatic] := 
  Module[{systemType, continuous, response, plot, estimatedDuration, 
    poles, realPoles, slowestPole, settlingTimeFactor = 4},
    
    systemType = Variables[TFM[[2]]][[1]];
    continuous = If[systemType == s, True, False];

    If[duration === Automatic,
      poles = TransferFunctionPoles[TFM];

      (* Select real parts of poles that are stable (negative real part) *)
      realPoles = Select[Flatten[Re /@ poles], # < 0 &];
      
      If[realPoles =!= {},
        (* Use the least negative real part to estimate the settling time *)
        slowestPole = Max[realPoles];
        estimatedDuration = settlingTimeFactor * Abs[1/slowestPole],
        (* If there are no stable real poles, use a longer default duration *)
        estimatedDuration = 100
      ],
      (* If duration is specified, use it *)
      estimatedDuration = duration
    ];

    (* Ensure that the estimatedDuration is a numeric value *)
    If[!NumericQ[estimatedDuration], estimatedDuration = 100];

    response = If[continuous, 
      First@OutputResponse[TFM, UnitStep[t], {t, 0, estimatedDuration}],
      First@OutputResponse[TFM, UnitStep[k], {k, 0, estimatedDuration}]
    ];

    plot = If[continuous, 
      Plot[response, {t, 0, estimatedDuration}, PlotRange -> All, 
        PlotStyle -> {Darker[Blue], Thick}, GridLines -> Automatic, 
        Frame -> True, Axes -> None, 
        FrameLabel -> {{"Response y(t)", None}, {"Time (sec)", "Step Response"}}, 
        ImageSize -> Medium],
      ListPlot[response, Filling -> None, PlotRange -> All, 
        DataRange -> {0, estimatedDuration}, AxesOrigin -> {0, 0}, 
        Frame -> True, Axes -> None, Joined -> True, 
        InterpolationOrder -> 0, PlotStyle -> {Darker[Red], Thick}, 
        GridLines -> Automatic, 
        FrameLabel -> {{"Response y(k)", None}, {"Time (sec)", "Step Response"}}, 
        ImageSize -> Medium]
    ];

    plot
  ];
```

### **Usage Example**

To utilize this function, first define a transfer function model, and then call **`PlotStepResponse`** with the model and the desired time duration.

```mathematica
TF[model_] := TransferFunctionModel[model, s];
Gs = TF[5/(s^2 + s + 5)];
PlotStepResponse[Gs, 15]
```

![Untitled](My%20Control%20Library%20for%20Mathematica%20b2e7b5f44bd440f2ae418f9ef75bddde/Untitled.png)

```mathematica
TF[model_] := TransferFunctionModel[model, s];
Gs = TF[5/(s^2 + s + 5)];
PlotStepResponse[Gs]
```

![Untitled](My%20Control%20Library%20for%20Mathematica%20b2e7b5f44bd440f2ae418f9ef75bddde/Untitled%201.png)