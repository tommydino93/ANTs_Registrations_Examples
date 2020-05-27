### Apply antsRegistrationSyNQuick.sh when patient has lesion (e.g. tumor)

Suppose we have a nifti T1-weighted sequence called `sub-xyz_T1w.nii.gz` with a big portion of the brain which has a lesion.
And suppose we want to register the MNI reference atlas `mni_icbm152_t1_tal_nlin_asym_09c.nii` to `sub-xyz_T1w.nii.gz`


If we directly apply **antsRegistrationSyNQuick.sh** to extract the registration parameters, we might not obtain a satisfying registration.
Instead, we can:
1) Segment the lesion (e.g. tumor) area in the brain with ITK-snap, thus obtaining a binary volume `lesion_mask_sub_xyz.nii.gz` that has 1s
in the lesion voxels and 0s outside.

2) Compute the inverse of this volume with:
```sh
ImageMath 3 inclusive_mask_sub_xyz.nii.gz Neg lesion_mask_sub_xyz.nii.gz
```
Now, the output volume `inclusive_mask_sub_xyz.nii.gz` has 0s in the lesion voxels and 1s outside of it.

3) Finally apply `antsRegistrationSyNQuick.sh` making use of the `-x` (mask) argument:
```sh
antsRegistrationSyNQuick.sh -m /path/to/mni_icbm152_t1_tal_nlin_asym_09c.nii -f /path/to/sub-xyz_T1w.nii.gz -t s -o /path/to/output_dir/out_MNI_2_T1_ -x /path/to/inclusive_mask_sub_xyz.nii.gz

```
where:
* **-m** stands for "moving" volume
* **-f** stands for "fixed" volume
* **-t** lets us decide the type of registration (e.g. "s" performs a rigid + affine + deformable syn)
* **-o** stands for "output" directory
* **-x** is used for selecting the mask

In general, mask values above 0.5 are considered non-outlier or non-lesioned regions.
