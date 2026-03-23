# VLA Notes

原文内容按主题拆分为 5 份笔记，正文只做 Markdown 排版与可视化组织，不改原意。

```mermaid
flowchart TD
    A[VLA：适合做 pick and place 的原因]
    B[纯 VLA 路线]
    C[WAM / World Action Model 路线]
    D[RL / Sim-to-Real 路线]
    E[混合路线]

    A --> B
    B --> E
    C --> E
    D --> E
```

- [pick_and_place.md](./pick_and_place.md)
- [pure_vla.md](./pure_vla.md)
- [wam.md](./wam.md)
- [rl_sim2real.md](./rl_sim2real.md)
- [hybrid_route.md](./hybrid_route.md)
