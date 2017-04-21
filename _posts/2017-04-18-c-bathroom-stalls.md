---
title: C. Bathroom Stalls
tags:
  - Partition
  - Devide and Conquer
  - Dynamic Programming
  - Python
categories:
  - Code Jam
  - 2017
  - Qualification
problem: TODO url here
---

ลองพิจารณาห้องน้ำที่มีจำนวนห้องย่อย $N=13$ (ไม่นับรวมหัวท้าย) และมีคนต้องการเข้าห้องน้ำ $K=6$ คน แผนภาพของการเพิ่มคนเข้าห้องน้ำทีละคนจะมีหน้าตาดังนี้

    k=0   .............
    k=1   ......0......     left=5  right=5
    k=2   ..0...0......     left=2  right=3
    k=3   ..0...0..0...     left=2  right=3
    k=4   ..0.0.0..0...     left=1  right=1
    k=5   ..0.0.0..0.0.     left=1  right=1
    k=6   0.0.0.0..0.0.     left=0  right=1

ข้อสังเกตประการแรกก็คือ ถ้าเราแบ่ง partition ของห้องน้ำว่างๆ ที่อยู่ติดกัน (สัญลักษณ์ `.` ติดกัน) แล้วสลับที่แต่ละ partition ไปมา ไม่ว่ายังไงคำตอบก็จะไม่เปลี่ยน (เพราะตัวโจทย์ไม่ได้ถามหาตำแหน่งของคนสุดท้าย แต่ถามว่าเหลือพื้นที่ว่างด้านซ้าย-ขวาเท่าไหร่) เช่น ตอนที่ `k=3` เราอาจจัดเรียง partition ตามจำนวนห้องน้ำว่างจากน้อยไปมากได้ดังนี้

    k=3   ..0...0..0...     original
    k=3   ..0..0...0...     sorted

เมื่อต้องคำนวณ `k=4` ก็ดูเพียง partition ขวาสุดก็พอแล้ว หลังจากใส่คนเข้าห้องน้ำใน partition นั้นสำเร็จ ก็เรียง partition ทั้งหมดอีกรอบ เช่นนี้

    k=4   ..0..0...0.0.     insert right partition
    k=4   .0.0..0..0...     sorted

ซึ่งนำไปสู่ข้อสังเกตที่สองของเรา ว่าเราสามารถลดรูปข้อมูล partition ให้เหลือเพียงแค่ตัวเลขได้ เช่นนี้

    k=0   partitions=[13]
    k=1   partitions=[6, 6]
    k=2   partitions=[2, 3, 6]
    k=3   partitions=[2, 2, 3, 3]
    k=4   partitions=[1, 1, 2, 2, 3]
    k=5   partitions=[1, 1, 1, 1, 2, 2]
    k=6   partitions=[0, 1, 1, 1, 1, 1, 2]

ยิ่งไปกว่านั้น เนื่องจากตัวเลขแต่ละ partition มักซ้ำกัน เราสามารถเก็บข้อมูลแบบ key-value โดยให้ key คือขนาดของ partition และ value คือจำนวนของ partition ที่มีขนาดนั้น เพื่อเพิ่มความเร็วของการเพิ่มคนคราวละมากกว่าหนึ่งคนได้อีกด้วย ดังนี้

    k=0   partitions={13: 1}
    k=1   partitions={6: 2}
    k=3   partitions={2: 2, 3: 2}
    k=5   partitions={1: 4, 2: 2}
    k=6   partitions={0: 1, 1: 5, 2: 1}

จากโจทย์ จำนวนห้องน้ำที่มากที่สุดคือ $N=10^{18}$ การแตก partition ใหญ่หนึ่งครั้งจะทำให้เกิด partition เล็กที่มีขนาดประมาณครึ่งหนึ่งของขนาดเดิม แต่เนื่องจาก partition จะหยุดแตกตัวต่อที่ขนาดศูนย์ ดังนั้นขอบเขตบนของการแตก partition จะเป็น $O(\log{N})$ ครั้ง ซึ่งเป็นเวลาที่ยอมรับได้สำหรับการแก้โจทย์ปัญหาข้อนี้

ด้านพื้นที่จัดเก็บ แต่ละครั้งที่ partition ใหญ่แตกตัว จะทำให้เกิด partition เล็กได้มากสุด 2 อัน (คือทั้งสองมีขนาดไม่เท่ากัน) ดังนั้นพื้นที่จัดเก็บที่ต้องใช้ก็จะเป็น $O(\log{N})$ เช่นกัน

---

ตัวอย่างคำตอบที่เขียนด้วยภาษา Python

``` python
from collections import defaultdict

def is_odd(number):
    return number % 2 == 1

def split_partition(number):
    half = number // 2
    if is_odd(number):
        return half, half
    else:
        return half-1, half

def find_stalls(stalls, persons):
    partitions = defaultdict(int, {stalls: 1})
    while persons:
        size = max(partitions.keys())
        capacity = partitions[size]
        del partitions[size]
        left, right = split_partition(size)
        if persons <= capacity:
            return right, left
        persons -= capacity
        partitions[left] += capacity
        partitions[right] += capacity

def main():
    for case in range(int(input())):
        n, k = [int(n) for n in input().split()]
        print('Case #{}: {} {}'.format(case+1, *find_stalls(n, k)))

if __name__ == '__main__':
    main()

```
