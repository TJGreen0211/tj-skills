# Joint Constraints - Bevy Rapier Plugin (0.30)

Source: https://rapier.rs/docs/user_guides/bevy_plugin/joint_constraints

## Two Approaches

| Aspect | Reduced-Coordinates (MultibodyJointSet) | Constraints-Based (ImpulseJointSet) |
|--------|----------------------------------------|-------------------------------------|
| Joint violations | Impossible | Possible if solver doesn't converge |
| Time steps | Moderately large OK | Large steps may explode |
| Large assemblies | Stable | Easily break without many iterations |
| Add/remove joints | Slow | Fast |
| Joint forces | Not computed | Always computed, retrievable |
| Topology | Tree structure only | Any graph structure |

## When to Use Each

- **Robotics:** Reduced-coordinates (accuracy, stability, IK support)
- **Video games (small assemblies, frequent add/remove):** Constraints-based
- **Closed loops (necklaces, chains):** Combine both approaches

## Multibodies

Multibodies use reduced-coordinates. Created similarly to impulse joints but inserted into `MultibodyJointSet`.

## Combining Both (Closed Loops)

For assemblies with loops (not tree-structured):

1. Define a multibody from a spanning-tree of the graph
2. Create joint constraints (ImpulseJoint) for edges not in the spanning tree ("loop-closing constraints")

Example: A necklace with 5 pearls and 5 ball joints:
- Create 5 multibody links with 4 BallJoints (spanning tree)
- Add 1 BallConstraint (ImpulseJoint) between first and last link to close the loop
