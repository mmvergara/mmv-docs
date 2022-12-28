# Image Proccessing MMV-Dex

* CL = compression level <br/>
* Image used = [Sample Image 151kb](https://i.ibb.co/V24xjQT/kazu.png)

##### Packages Used
1. [Imagescript](https://deno.land/x/imagescript@1.2.15/mod.ts) `deno`
2. [Sharpjs](https://www.npmjs.com/package/sharp)
3. [Browser Image Compression](https://www.npmjs.com/package/browser-image-compression)


|  | Supabase Server | Vercel Server |
| ------------------------------------------------------- | --------------- | ------------- |
| **No Compression**                                      | 138.11 KB       | 95.46 KB      |
| **Client Side Compression**                             | 113.97 KB       | 94.69 KB      |
| **Server Side Compression**                             | 127.85 KB       | 26.65 KB      |

|                             | Supabase Server                             | Vercel Server                          |
| --------------------------- | ------------------------------------------- | -------------------------------------- |
| **No Compression**          | Imagescript **CL1**                         | Sharpjs **CL1**                        |
| **Client Side Compression** | Browser Image Compression **CL9** + Imagescript **CL1** | Browser Image Compression **CL9** + Sharpjs **CL1**|
| **Server Side Compression** | Imagescript **CL9**                         | Sharpjs **CL9**                        |
