# RISC-V SOC Week 8
# Post route STA - VSDBabySoC

## Last routing ended in Congestion

⚠️ Routing did not complete successfully

Global routing encountered congestion at multiple regions.

Final overflow reported (example):

```
[ERROR GRT-0116] Global routing finished with congestion.
```

## Cause for Congestion 
Your design had pins placed on a metal layer that the router could not physically reach, due to:
- Macro pin OBS layers
- Wrong LEF pin definitions
- Access blockage
- Incorrect pin layer in the IP or hard macro
The avsdac.lef file is changed to overcome the routing congestion problem.
![routing](assets/routing_crct.jpg)

## Final GDSII outputs

