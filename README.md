# My 3D Zen Ball Game (ชื่อโปรเจกต์ของคุณ)

โปรเจกต์เกม 3D ที่ได้รับแรงบันดาลใจจาก The Line Zen สร้างด้วย Godot Engine 4.x โดยมีเป้าหมายเพื่อสร้างเกมที่เล่นเพลิน มีระบบฟิสิกส์ที่สนุก และมีเครื่องมือสร้างด่านให้ผู้เล่นได้สร้างสรรค์ผลงานของตัวเอง

---

## Phase 0: การเตรียมตัวและวางรากฐาน (สำคัญที่สุด)

### 1. เครื่องมือที่ต้องเตรียม (ทั้งหมดฟรี)
- **Game Engine:** [Godot Engine 4.x](https://godotengine.org/) (เวอร์ชัน Standard สำหรับ GDScript)
- **3D Modeling:** [Blender](https://www.blender.org/)
- **Code Editor (แนะนำ):** [Visual Studio Code (VS Code)](https://code.visualstudio.com/) พร้อม Extension ของ Godot
- **เครื่องมือจัดการโปรเจกต์/จดบันทึก:**
  - [Trello](https://trello.com/)/[Asana](https://asana.com/) (สำหรับ Task Management)
  - [Notion](https://www.notion.so/)/[Obsidian](https://obsidian.md/) (สำหรับ Game Design Document)
- **Version Control:** [GitHub Desktop](https://desktop.github.com/) (สำหรับจัดการเวอร์ชันของโปรเจกต์)

### 2. แนวคิดที่ต้องปรับ (Mindset)
- **เริ่มจากเล็กที่สุด (Start Small):** สร้าง Prototype ที่มีแค่ฟังก์ชันหลักๆ ก่อน
- **ทำซ้ำและปรับปรุง (Iterate):** สร้าง -> ทดลอง -> ปรับปรุง -> วนไปเรื่อยๆ
- **ความล้มเหลวคือการเรียนรู้:** Error คือเรื่องปกติของการพัฒนาเกม

---

## Phase 1: สร้างตัวต้นแบบ (Core Gameplay Loop)

**เป้าหมาย:** สร้างเกมที่เล่นได้แบบพื้นฐานที่สุด

1.  **ตั้งค่าโปรเจกต์ Godot:** สร้างโปรเจกต์ใหม่, Renderer `Forward+`, สร้างซีนหลัก `Main.tscn`
2.  **สร้างผู้เล่น (ลูกบอล):**
    - สร้างซีนใหม่ `player.tscn` ด้วยโหนด `CharacterBody3D`
    - เพิ่ม `MeshInstance3D` (ใช้ `SphereMesh`) และ `CollisionShape3D` (ใช้ `SphereShape3D`)
3.  **การควบคุมด้วยเมาส์ (แบบหน่วงๆ):**
    - สร้าง Script `player.gd` และใส่โค้ดควบคุมการเคลื่อนที่, การแดช, และการชน
    ```gdscript
    # player.gd
    extends CharacterBody3D

    # ค่าความเร็วในการเคลื่อนที่
    @export var speed = 10.0
    # ค่าน้ำหนัก หรือความหน่วง (0.0 คือตามเมาส์ทันที, 1.0 คือไม่ขยับเลย)
    @export var weight = 0.1 
    # ค่าความแรงในการแดช
    @export var dash_strength = 30.0

    var mouse_world_position = Vector3.ZERO

    func _physics_process(delta):
        # --- ส่วนที่ 1: หาตำแหน่งของเมาส์ในโลก 3D ---
        var mouse_pos = get_viewport().get_mouse_position()
        var camera = get_viewport().get_camera_3d()
        
        var from = camera.project_ray_origin(mouse_pos)
        var dir = camera.project_ray_normal(mouse_pos)
        
        var plane = Plane(Vector3.UP, global_position.y) 
        var intersection = plane.intersects_ray(from, dir)
        
        if intersection:
            mouse_world_position = intersection

        # --- ส่วนที่ 2: การเคลื่อนที่แบบหน่วงๆ (Interpolation) ---
        var direction = global_position.direction_to(mouse_world_position)
        var desired_velocity = direction * speed
        velocity = velocity.lerp(desired_velocity, weight)
        
        # --- ส่วนที่ 3: การแดช (Dash) ---
        if Input.is_action_just_pressed("dash"):
            velocity = velocity.normalized() * dash_strength

        move_and_slide()
        
        # --- ส่วนที่ 4: การชนและเด้งกลับ (Knockback) ---
        if get_slide_collision_count() > 0:
            var collision = get_slide_collision(0)
            var knockback_distance = 5.0
            var knockback_vector = collision.get_normal() * knockback_distance
            
            global_position += knockback_vector
            print("Ouch! I hit something!")
    ```
4.  **สร้างฉากและสิ่งกีดขวาง:**
    - ในซีน `Main` เพิ่ม Player, พื้น (`MeshInstance3D` + `PlaneMesh`), และกำแพง (`StaticBody3D` + `BoxMesh` + `BoxShape3D`)
    - เพิ่ม `Camera3D` และ `DirectionalLight3D`
5.  **ตั้งค่า Input Map:** ไปที่ `Project -> Project Settings -> Input Map` เพิ่ม Action ชื่อ `dash` แล้วผูกกับ `Left Mouse Button`

---

## Phase 2: การสร้างโลกและกราฟิก

1.  **Blender to Godot Workflow:**
    - สร้างโมเดลแบบ Low-poly
    - กด `Ctrl+A` -> `All Transforms` ก่อน Export
    - Export เป็น `.glb` (glTF Binary) โดยติ๊ก `Selected Objects` และ `+Y Up`
    - ลากไฟล์ `.glb` เข้าโปรเจกต์ Godot
2.  **สร้างเครื่องมือสร้าง Map (Map Editor Prototype):**
    - สร้าง `MeshLibrary` จากชิ้นส่วนโมเดลต่างๆ (ทางตรง, ทางโค้ง, กำแพง)
    - ใช้โหนด `GridMap` ในซีนหลัก แล้วลาก `MeshLibrary` ไปใส่ เพื่อเริ่ม "วาด" ด่าน

---

## Phase 3: การขัดเกลาและเพิ่มประสิทธิภาพ

1.  **เพิ่ม "ความสนุก" (Game Feel & Juice):**
    - เอฟเฟกต์ไฟฟ้าช็อต (`GPUParticles3D`)
    - เสียง (`AudioStreamPlayer3D`)
    - บรรยากาศด้วย `WorldEnvironment` (เปิด `Glow`, `SSAO`)
2.  **การลดการกินสเปค (Optimization):**
    - ใช้ระบบ `Occlusion Culling` โดยการวาง `OccluderInstance3D` เพื่อบังวัตถุที่กล้องมองไม่เห็น

---

## Phase 4: ระบบเกมระยะยาว (Metagame & Features)

- **ระบบ UI:** สร้างเมนู, คะแนน ด้วยโหนด `Control`
- **ระบบ Achievement:** สร้าง `Autoload (Singleton)` เพื่อเก็บสถิติผู้เล่น
- **ระบบปลดล็อค:** ปลดล็อค Material หรือสกิลเมื่อทำ Achievement สำเร็จ
- **เครื่องมือสร้าง Map สำหรับผู้เล่น:** สร้าง UI ครอบระบบ `GridMap`
- **Steam Workshop:** เชื่อมต่อกับ Steamworks API (ขั้นสูง)

---

## Project Checklist

### Milestone 1: สร้าง Prototype ที่เล่นได้
- [ ] ติดตั้ง Godot, Blender, VS Code, Git
- [ ] สร้างโปรเจกต์ Godot
- [ ] สร้างซีน Player (CharacterBody3D + Mesh + Collision)
- [ ] เขียนโค้ดควบคุม Player ด้วยเมาส์ (หาตำแหน่งเมาส์ใน 3D)
- [ ] เขียนโค้ดเคลื่อนที่แบบหน่วง (Lerp)
- [ ] สร้างซีน Level (พื้น, กำแพง StaticBody3D)
- [ ] เพิ่ม Player, Camera, Light ใน Level
- [ ] เขียนโค้ดการชนและเด้งกลับ (Knockback)
- [ ] เขียนโค้ดการแดช (Dash)
- [ ] ทดสอบจนรู้สึกว่า "พอเล่นได้"

### Milestone 2: สร้างเนื้อหาและโลกของเกม
- [ ] เรียนรู้พื้นฐานการ Export โมเดลจาก Blender เป็น .glb
- [ ] สร้างโมเดลชิ้นส่วนด่าน 3-4 แบบ (ทางตรง, ทางเลี้ยว, กำแพง)
- [ ] สร้าง MeshLibrary จากโมเดลเหล่านั้น
- [ ] ใช้ GridMap สร้างด่านทดสอบ 1 ด่าน
- [ ] ปรับปรุงแสงเงาเบื้องต้นด้วย WorldEnvironment (Glow, SSAO)

### Milestone 3: ขัดเกลาให้เป็น "เกม"
- [ ] เพิ่มเอฟเฟกต์ไฟฟ้าช็อต (Particle)
- [ ] เพิ่มเสียงประกอบ (แดช, ชน, เพลงพื้นหลัง)
- [ ] สร้าง UI พื้นฐาน (หน้าจอเริ่มเกม, จบเกม)
- [ ] สร้างระบบนับคะแนน (เช่น นับระยะทาง)
- [ ] เริ่มทำ Optimization ด้วย Occlusion Culling

### Milestone 4: ฟีเจอร์ระยะยาว (Post-Launch / Major Update)
- [ ] ออกแบบระบบ Achievement
- [ ] เขียนโค้ดระบบ Achievement (ใช้ Autoload/Singleton)
- [ ] ออกแบบและสร้างระบบปลดล็อค (เช่น เปลี่ยนสีลูกบอล)
- [ ] ออกแบบ UI สำหรับเครื่องมือสร้างด่านของผู้เล่น
- [ ] พัฒนาเครื่องมือสร้างด่านสำหรับผู้เล่น
- [ ] ศึกษาและติดตั้ง GodotSteam
- [ ] เชื่อมต่อระบบ Save/Load ด่านกับ Steam Workshop
