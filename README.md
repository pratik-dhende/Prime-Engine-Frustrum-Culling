# Prime Engine Frustrum Culling

## Demo Link: 
[https://drive.google.com/file/d/1TWowICk02NKH9F_bIC43cA_bnkLpScQK/view?usp=sharing](https://drive.google.com/file/d/1TWowICk02NKH9F_bIC43cA_bnkLpScQK/view?usp=sharing)

## Implementation details
1. Added 2 member variables `m_localMinAABBPoint` and `m_localMaxtAABBPoint` to `Mesh.h` to keep the track of the minimum and maximum point of the AABB.

1. Extracted the mesh positions from the `PositionBufferCPU` and calculate the minimum mesh position and maximum mesh position for each mesh in the `MeshManager::getAsset()` function and stored it in the `Mesh::m_localMinAABBPoint` and `Mesh::m_localMaxtAABBPoint`.

1. In the `GameThreadJob.cpp`, I extracted the `Mesh::m_localMinAABBPoint` and `Mesh::m_localMaxtAABBPoint` for each `MeshInstance`, converted them to to world space, calculated all the 8 points of AABB and used `DebugRenderer::Instance()->createLineMesh()` to draw the AABB for each `MeshInstance`.

1. Created a `struct PE::Components::Plane` which represents a plane in the form of `Ax + By + Cz + D = 0`.

1. The constants of the plane are calculated using a point on the plane and the normal of the plane.

1. The test of a point for a plane is done in the following way:
    1. `Ax + By + Cz + D < 0`, then the point is inside the plane.
    1. `Ax + By + Cz + D = 0`, then the point is on the plane.
    1. `Ax + By + Cz + D > 0`, then the point is outside the plane.

1. Created a `struct PE::Components::Frustrum` containing 6 planes and 4 member variables for storing near plane distance, far plane distance, vertical field of view, horizontal field of view.

1. The construction of frustrum was done in the following steps.
    1. For the left plane, it's camera is rotated towards left, it's `-right` vector is taken from the resulting transform and used as the normal, and camera's position is taken as the point on the plane.
    1. For the right plane, it's camera is rotated towareds right, it's `right` vector is taken from the resulting transform and used as the normal, and camera's position is taken as the point on the plane.
    1. For the top plane, it's camera is rotated towards up, it's `up` vector is taken from the resulting transform and used as the normal, and camera's position is taken as the point on the plane.
    1. For the bottom plane, it's camera is rotated towards bottom, it's `-up` vector is taken from the resulting transform and used as the normal, and camera's position is taken as the point on the plane.
    1. For the near plane, the camera's `-forward` vector is taken as the normal, the camera's position is translated in the forward direction by the near plane distance and is used as the point on the plane.
    1. For the far plane, the camera's `forward` vector is taken as the normal, the camera's position is translated in the forward direction by the far plane distance and is used as the point on the plane.

1. A point is inside the frustrum if it is inside all of the frustrum planes.

1. For an AABB, if none of it's 8 points fall under the fustrum, then it is culled out.

1. This camera frustrum is created inside the `CameraSceneNode::do_CALCULATE_TRANSFORMATIONS()` and is stored as a member variable of `CameraSceneNode` as `Frustrum m_cameraFrustrum`.

1. Inside the `struct Event_GATHER_DRAWCALLS`, `Frustrum* m_frustum` member variable is created to store the camera's frustrum to pass it on to the `SingleHandler_DRAW::do_GATHER_DRAWCALLS()` function.

1. In the `GameThreadJob.cpp`, this member variable is assigned the camera's frustrum.

1. When the `Event_GATHER_DRAWCALLS` event reaches the `SingleHandler_DRAW::do_GATHER_DRAWCALLS()`, the frustrum is extracted.

1. This frustrum then goes over each of the mesh instances, checks whether the AABB for the mesh instance lies within the frustrum or not and sets the `MeshInstance::m_culledOut` accordingly.
