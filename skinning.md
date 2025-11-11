glTF 的 skin.skeleton 和 Unity 的 root bone 究竟是什么？

[The rootbone has nothing to do with the actual skinning of the SkinnedMeshRenderer, It has been added for the root motion feature of the the mecanim system.](https://discussions.unity.com/t/is-it-possible-to-update-a-skinned-mesh-renderers-bones/199255/2)

## com.unity.entities.graphics [0.8.0] - 2020-08-04
* Changed SkinnedMeshRendererConversion to take the **RootBone** into account. The render entities are now parented to the **RootBone** entity instead of the SkinnedMeshRenderer GameObject entity. As a result the RenderBounds will update correctly when the **root bone** is transformed.
* Changed SkinnedMeshRendererConversion to compute the SkinMatrices in SkinnedMeshRenderer's **root bone** space instead of worldspace.