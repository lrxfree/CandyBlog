---
title: Unity中 第一人称控制与第三人称控制 的实现心得
date: 2025-04-10 17:14:00
tags: Unity
cover: https://s21.ax1x.com/2025/04/21/pE5uMnS.jpg
---

# Unity中 第一人称控制与第三人称控制 的实现心得

## 一、直接手写实现

- ### Unity 旋转方面的基础知识

1. **欧拉角（Euler Angles）**

   欧拉角是一种用三个角度（通常是绕 X、Y、Z 轴的旋转）来表示三维空间中旋转的方法。

   - **Yaw（偏航）**：绕 Y 轴的旋转（左右旋转）。

   - **Pitch（俯仰）**：绕 X 轴的旋转（上下旋转）。

   - **Roll（翻滚）**：绕 Z 轴的旋转（自身旋转）。

   **使用场景**

   - **直观**：欧拉角非常直观，适合简单的旋转和调试。

   - **输入和输出**：通常用于输入和输出旋转角度，因为它们易于理解和修改。

   **缺点**

     - **万向节锁（Gimbal Lock）**：当两个旋转轴对齐时，会丢失一个自由度。例如，当 Pitch 达到 90 度时，Yaw 和 Roll 的效果会重叠。

     - **插值困难**：欧拉角的插值（如在两个旋转之间平滑过渡）比较复杂。

   ```c#
   transform.eulerAngles = new Vector3(0, 90, 0); // 绕 Y 轴旋转 90 度
   ```

2. **四元数（Quaternion）**

   四元数是一种数学结构，用于表示三维空间中的旋转。它由四个分量组成：`w`、`x`、`y` 和 `z`

   - **避免万向节锁**：四元数不会出现万向节锁问题，因此在复杂旋转中更稳定。

   - **高效插值**：四元数支持高效的插值方法（如 `Slerp` 和 `Lerp`），用于实现平滑的旋转过渡。

   - **计算复杂**：四元数的数学运算相对复杂，但 Unity 提供了内置方法来简化操作。

   **使用场景**

     - **内部表示**：Unity 内部使用四元数来表示旋转。

     - **复杂旋转**：适合复杂的旋转操作和插值。

   ```C#
   Quaternion rotation = Quaternion.Euler(0, 90, 0); // 从欧拉角创建四元数
   transform.rotation = rotation; // 应用旋转
   ```

3. #### rotation

   - **`rotation`** 是一个 `Quaternion` 类型的属性，表示物体的旋转状态。

   - 它可以用来获取或设置物体的旋转。

   ```c#
   	transform.rotation = Quaternion.Euler(0, 90, 0); // 将物体绕 Y 轴旋转 90 度
   ```

4. **`localRotation`**
   - **`localRotation`** 也是 `Quaternion` 类型，表示物体在本地坐标系中的旋转。

   - 与 **rotation** 不同，**localRotation** 是相对于父物体的旋转。

   ```c#
   transform.localRotation = Quaternion.Euler(0, 90, 0); // 设置本地旋转
   ```

5. **eulerAngles**

     - **`eulerAngles`** 是一个 `Vector3` 类型的属性，表示物体的旋转角度（欧拉角），单位是度。

     - 它可以用来获取或设置物体的旋转角度。

   ```c#
   transform.eulerAngles = new Vector3(0, 90, 0); // 将物体绕 Y 轴旋转 90 度
   ```

6. **`LookRotation`**

     - **`Quaternion.LookRotation`** 是一个静态方法，用于创建一个朝向指定方向的旋转。

     - 方法签名：`public static Quaternion LookRotation(Vector3 forward, Vector3 upwards = Vector3.up);`

       - **`forward`**：物体的前向方向。

       - **`upwards`**：物体的向上方向，默认为 `Vector3.up`

     ```csharp
     Quaternion targetRot = Quaternion.LookRotation(movement); // 创建朝向 movement 方向的旋转
     ```

7. **`Slerp`**

  - **`Quaternion.Slerp`** 是一个静态方法，用于在两个旋转之间进行球面线性插值。
  
  - 方法签名：`public static Quaternion Slerp(Quaternion a, Quaternion b, float t);`
  
    - **`a`**：起始旋转。
  
    - **`b`**：目标旋转。
  
    - **`t`**：插值参数，范围通常在 `[0, 1]` 之间。
  
    ```csharp
    transform.rotation = Quaternion.Slerp(transform.rotation, targetRot, 10f * Time.deltaTime); // 平滑旋转到目标方向
    ```

8. **`Lerp`**

  - **`Quaternion.Lerp`** 是另一个静态方法，用于在两个旋转之间进行线性插值。
  
  - 与 `Slerp` 不同，`Lerp` 的插值是线性的，而不是球面的。
  
  - 方法签名：`public static Quaternion Lerp(Quaternion a, Quaternion b, float t);`
  
   ```csharp
  transform.rotation = Quaternion.Lerp(transform.rotation, targetRot, Time.deltaTime); // 线性插值旋转
   ```
  
9. **`Euler`**

  - **`Quaternion.Euler`** 是一个静态方法，用于从欧拉角创建一个旋转。
  
  - 方法签名：`public static Quaternion Euler(Vector3 euler);`
  
   ```csharp
  Quaternion rotation = Quaternion.Euler(0, 90, 0); // 创建绕 Y 轴旋转 90 度的旋转
   ```
  
10. **`ProjectOnPlane`**

  - **`Vector3.ProjectOnPlane`** 是一个静态方法，用于将一个向量投影到一个平面上。
  
  - 方法签名：`public static Vector3 ProjectOnPlane(Vector3 vector, Vector3 planeNormal);`
  - 将向量 `vector` 投影到由法向量 `planeNormal` 定义的平面上，返回投影后的向量。
  
  ```csharp
  Vector3 camForward = Vector3.ProjectOnPlane(_thirdPersonCamera.transform.forward, Vector3.up).normalized; // 将摄像机前向向量投影到水平面
  ```
  
11. **`Rotate`**

   - **`Transform.Rotate`** 是控制物体旋转的核心方法之一。

   - 方法签名：`public void Rotate(Vector3 eulers, Space relativeTo = Space.Self);`

   ```csharp
   transform.Rotate(Vector3.up * 90); // 绕 Y 轴旋转 90 度
   ```

  注意**旋转坐标系的选择**

   - **`Space.Self`（默认）**：绕物体的**局部坐标系轴**旋转。

     ```c#
     ransform.Rotate(0, 90, 0); // 绕物体自身的Y轴旋转90度
     ```

   - **`Space.World`**：绕**世界坐标系轴**旋转。

     ```c#
     transform.Rotate(Vector3.up, 90, Space.World); // 绕世界Y轴旋转90度
     ```

  与直接设置旋转的区别

- **`Rotate` vs `rotation`/`eulerAngles`**：

    - Rotate 是增量操作（叠加旋转）
    
    - transform.eulerAngles = new Vector3(0, 90, 0) 是覆盖操作（直接设置绝对角度）。
12. **`Right` 和 `Forward`**

  **`Transform.right`** 和 **`Transform.forward`** 是属性，分别表示物体的右向和前向向量。


13. **`localEulerAngles`**

   **`localEulerAngles`** 是一个 `Vector3` 类型的属性，表示物体在本地坐标系中的旋转角度。

   ```csharp
   transform.localEulerAngles = new Vector3(0, 90, 0); // 设置本地旋转角度
   ```

14.  **`up`**

   - **`Transform.up`** 是一个属性，表示物体的上方向向量。

15. **`LookAt`**

   - **`Transform.LookAt`** 是一个方法，用于使物体朝向指定目标。

   - 方法签名：`public void LookAt(Vector3 worldPosition, Vector3 worldUp = Vector3.up);`

     **对比其他旋转方法**

    

   | 方法                    | **适用场景**                 | **特点**                       |
   | ----------------------- | ---------------------------- | ------------------------------ |
   | Transform.LookAt        | 快速让物体注视目标点         | 简单直接，适合快速实现基础功能 |
   | Quaternion.LookRotation | 需要手动控制旋转或插值       | 灵活性高，适合复杂逻辑         |
   | Rotate                  | 增量式旋转（如连续旋转动画） | 需注意坐标系和旋转顺序         |

   

16. **`Quaternion.identity`**

   - **`Quaternion.identity`** 是一个静态属性，表示没有旋转的单位四元数。

17. **`Angle`**

   - **`Quaternion.Angle`** 是一个静态方法，用于计算两个旋转之间的角度。

   - 方法签名：`public static float Angle(Quaternion a, Quaternion b);`

     
- ### 第一人称代码实现

**目录结构**

> Player - **[挂载 Character Controller]**
>
> > Firstperson Camera - **[挂载 FirstpersonCamera ]**

**脚本 `FirstpersonCamera `  > 基本属性**

```c#
public float mouseSensitivity = 50f;
private float xRotation = 0f;
private float yRotation = 0f;
    
public Transform charactorTransform;//角色Transform组件
    
private PlayerMoveControl playerMoveControl;//Input System
public PlayerControler playerControler;
    [Tooltip("旋转灵敏度")]
public float lookSensitivity = 20f;
```

**脚本 `FirstpersonCamera `  > 初始化**

```c#
private void Awake()
{
     Cursor.lockState = CursorLockMode.Locked;//鼠标光标隐藏
     Cursor.visible = false;
        
     playerMoveControl=new PlayerMoveControl();//Input System 初始化
     playerMoveControl.Enable();//Input System 激活【激活才能用】
        
}
```

**脚本 `FirstpersonCamera `  > LookControl**

```c#
private void LookControl()
{
    Vector2 mouse = playerMoveControl.Player.Look.ReadValue<Vector2>();

    float mouseX = mouse.x*mouseSensitivity*Time.deltaTime;
    float mouseY = mouse.y*mouseSensitivity*Time.deltaTime;
    
    //垂直旋转
    xRotation -= mouseY;
    xRotation = Mathf.Clamp(xRotation, -40f, 60f);//对上下视角的限制；
    //左右旋转
    yRotation += mouseX;
  
    //运用上下旋转,采用插值平滑移动
    transform.localRotation=Quaternion.Lerp(transform.localRotation,Quaternion.Euler(xRotation,0,0f),lookSensitivity*Time.deltaTime);

    
    if(!playerControler.isThirdPersion)  //运用左右旋转,采用插值平滑移动
        charactorTransform.localRotation=Quaternion.Lerp(charactorTransform.localRotation,Quaternion.Euler(0f,yRotation,0f),lookSensitivity*Time.deltaTime);
}
```


- 左右转动直接旋转角色；

- 上下转动旋转相机；

- ### 第三人称代码实现

**目录结构**

> Player  [挂载 Character Controller]
>
> > ThirdPersonCamera**[挂载 ThirdPersonCamera]**

**脚本 `ThirdPersonCamera `  > 基本属性**

```c#
PlayerMoveControl playerMoveControl;

[Header("Target Setting")]
public Transform target;
public float distance = 0.05f;
public float minVerticalAngle = -20f;
public float maxVerticalAngle = 40f;


[Header("Camera Settings")] 
public float damping = 0.1f;
[Tooltip("旋转速度")][Range(10,200)]public float rotateSpeed  = 20f;
public Vector3 cameraOffset = new Vector3(0f, 0f, 0f);
[Tooltip("摄像机需要检测的layer")]
public LayerMask cameraColliderLayer;
[Tooltip("检测到物体之后，规避的距离")]
public float collisionOffset = 0.1f;
[Tooltip("摄像机碰撞检测的范围")]
public float checkRadius = 0.1f;

private float _currentX;
private float _currentY;
private Vector3 _refVelocity; //平滑用
```

**脚本 `ThirdPersonCamera `  > 初始化**

```c#
void Start()
{
    playerMoveControl = new PlayerMoveControl();
    playerMoveControl.Enable();
}
```

**脚本 `ThirdPersonCamera `  > 获取鼠标移动**

```c#
private void UpdateCameraRotation()
{//旋转角计算
    _currentX += playerMoveControl.Player.Look.ReadValue<Vector2>().x * rotateSpeed * Time.deltaTime;
    _currentY -= playerMoveControl.Player.Look.ReadValue<Vector2>().y * rotateSpeed * Time.deltaTime;
    _currentY = Mathf.Clamp(_currentY, minVerticalAngle, maxVerticalAngle);

    if (float.IsNaN(_currentX)) _currentX = 0;
    if (float.IsNaN(_currentY)) _currentY = 0;
}
```

**脚本 `ThirdPersonCamera `  > 获取理论镜头位置**

```c#
private Vector3 GetIdealCameraPosition()
{
    Quaternion rotation = Quaternion.Euler(_currentY, _currentX, 0f);//z轴归零
    Vector3 direction = (rotation * Vector3.forward).normalized;
    Vector3 targetPosition = target.position - direction * distance + Vector3.up * 0.5f+ cameraOffset;

    return targetPosition;
}
```

**脚本 `ThirdPersonCamera `  >核心碰撞检测逻辑**

```c#
private void HandleCollision()
{
    Vector3 idealCameraPosition = GetIdealCameraPosition();
    RaycastHit hit;
    
    Vector3 rayDirection = (idealCameraPosition - target.position).normalized;
    float rayDistance = Vector3.Distance(target.position, idealCameraPosition);

    if (Physics.SphereCast(target.position + Vector3.up * 0.3f, checkRadius, rayDirection, out hit, rayDistance,
            cameraColliderLayer))
    {
        transform.position = hit.point + hit.normal * collisionOffset;
    }
    else
    {
        //🌈 平滑移动到理想位置
        transform.position = Vector3.SmoothDamp(transform.position, idealCameraPosition, ref _refVelocity, Time.deltaTime * damping);
    }
}
```

**脚本 `ThirdPersonCamera `  >应用最终镜头运动**

```c#
private void ApplyCameraMovement()
{
    Quaternion finalRotation = Quaternion.Euler(_currentY, _currentX, 0);
    //transform.rotation = Quaternion.Slerp(transform.rotation, finalRotation, 0.8f * Time.deltaTime);
    transform.rotation = finalRotation;
    transform.LookAt(target.position + Vector3.up * cameraOffset.y);
}
```

**脚本 `ThirdPersonCamera `  >应用最终镜头运动**

```c#
private void ApplyCameraMovement()
{
    Quaternion finalRotation = Quaternion.Euler(_currentY, _currentX, 0);
    //transform.rotation = Quaternion.Slerp(transform.rotation, finalRotation, 0.8f * Time.deltaTime);
    transform.rotation = finalRotation;
    transform.LookAt(target.position + Vector3.up * cameraOffset.y);
}
```

**脚本 `ThirdPersonCamera `  >切换该相机时重置相机位置**

```c#
public void ResetCameraPosition()
{
    _currentX = target.eulerAngles.y;
    _currentY = 17f;
    transform.position = GetIdealCameraPosition();
}
```

**脚本 `ThirdPersonCamera `  >调试可视化**

```c#
void OnDrawGizmosSelected()
{
    Gizmos.color = Color.cyan;
    Gizmos.DrawWireSphere(transform.position, checkRadius);
    if(target) Gizmos.DrawLine(target.position, GetIdealCameraPosition());
}
```

**脚本 `ThirdPersonCamera `  >最终调用**

```c#
private void LateUpdate()
{
    UpdateCameraRotation();
    HandleCollision();
    ApplyCameraMovement();
}
```

- LateUpdate：需要在所有游戏对象更新之后再执行的逻辑，需要基于当前帧的最终状态进行更新的逻辑

- 

- | 方法        | 执行时机       | 时间步长            | 典型使用场景             |
  | ----------- | -------------- | ------------------- | ------------------------ |
  | Update      | 每帧渲染之前   | Time.deltaTime      | 大多数游戏逻辑更新       |
  | FixedUpdate | 物理引擎更新时 | Time.fixedDeltaTime | 物理相关计算             |
  | LateUpdate  | 所有  执行后   | Time.deltaTime      | 相机跟随、特效等后续处理 |

