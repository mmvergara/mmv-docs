
# Posts

- User's can create and delete post's.
- All posts are public
- Admins can do CRUD on post's

## Image Processing

- CL = compression level <br/>
- Image used = [Sample Image 151kb](https://i.ibb.co/V24xjQT/kazu.png)
- All images are converted png ( bad implementation )
> Converting to png results into more image file size 🤦🤦🤦 
#### Packages Used

1. [Imagescript](https://deno.land/x/imagescript@1.2.15/mod.ts) `deno`
2. [Sharpjs](https://www.npmjs.com/package/sharp)
3. [Browser Image Compression](https://www.npmjs.com/package/browser-image-compression)

|                             | Supabase Server | Vercel Server |
| --------------------------- | --------------- | ------------- |
| **No Compression**          | 138.11 KB       | 95.46 KB      |
| **Client Side Compression** | 113.97 KB       | 94.69 KB      |
| **Server Side Compression** | 127.85 KB       | 26.65 KB      |

|                                   | Supabase Server                                                     | Vercel Server                                                   |
| --------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------- |
| **No Compression**                | Imagescript <br/> **CL1**                                           | Sharpjs <br/> **CL1**                                           |
| <br/> **Client Side Compression** | Browser Image Compression <br/> **CL9** + Imagescript <br/> **CL1** | Browser Image Compression <br/> **CL9** + Sharpjs <br/> **CL1** |
| **Server Side Compression**       | Imagescript <br/> **CL9**                                           | Sharpjs <br/> **CL9**                                           |

## Client/Server Side Validation
Both in the Server and Client, this project uses the combination of Yup and Formik to validate post's data

### Image Validation
Different validation methods for different packages are applied this is the one used for for formidable which will validate the file type and file size
```js
// import { File as formidableFile } from "formidable";
// file is temporarily typed as any Formidable having issues with nextjs
export const formidableFileValidation = (file: any, allowedFileTypes: string[]) => {
  let error: { message: string } | null = null;
  let message = "";

  // Check if it's a PersistentFile formidable file
  if (!file?.mimetype) message += "Invalid received data is not a file";

  // Check file type
  if (file.mimetype) {
    const mimetype = file.mimetype;
    if (!allowedFileTypes.some((fileType) => fileType === mimetype.split("/").pop())) {
      message += "Invalid file type";
    }
  }

  // Max 10MB
  // Client feedback shows 8MB making extra room for the server side image processing
  if (file.size > 8_000_000) message += "File exceeds maximum size (8MB)";
  if (!!message) error = { message };

  return { error };
};
```

[Browser Image Compression](https://www.npmjs.com/package/browser-image-compression) offer's client side validation
```js
if (compressionMethod === "client") imgFile = await imageCompression(image, { maxSizeMB: 10 });
```


## Keyword Analysis

### Fetching
fetching post's is dead simple by using the supabase sdk and we are only selecting the columns that we need for the keyword analysis `id`, `description`, `author`
```js
  const fetchPostDescriptions = async (supabase: SupabaseClient<DatabaseTypes>) => {
    return await supabase.from("posts").select("id,description,author");
  };
```

### Analyzation
this function will take the data and the search query string and will output the following
```js
  {
    totalPosts,
    total_OccurenceOf_X_InDesc,
    total_OccurenceOf_X_InEachPostDescription_WithId,
    total_UsersUsing_X_inTheirDesc: seenAuthors.size,
  };
```
```js
// Refer to X as the "search query string"
export const postsKeywordAnalysis = (
  posts: { id: number; description: string; author: string }[] | null,
  searchString: string
) => {
  if (!posts || searchString.length === 0) return null;
  const totalPosts = posts.length;
  let total_OccurenceOf_X_InDesc = 0;
  let total_OccurenceOf_X_InEachPostDescription_WithId: { id: number; searchStringOccurenceCount: number }[] = [];

  // We are using a set so the users id won't be duplicated
  const seenAuthors = new Set<string>();

  for (let i = 0; i < posts.length; i++) {
    const descriptionHasSearchString = posts[i].description.toLowerCase().includes(searchString.toLowerCase());

    if (descriptionHasSearchString) {
      const occurenceOfXinTheCurrentDescription = countOccurrences(posts[i].description, searchString);
      total_OccurenceOf_X_InDesc += occurenceOfXinTheCurrentDescription;
      total_OccurenceOf_X_InEachPostDescription_WithId.push({
        id: posts[i].id,
        searchStringOccurenceCount: occurenceOfXinTheCurrentDescription,
      });
    }
    if (descriptionHasSearchString) seenAuthors.add(posts[i].author);
  }

  return {
    totalPosts,
    total_OccurenceOf_X_InDesc,
    total_OccurenceOf_X_InEachPostDescription_WithId,
    total_UsersUsing_X_inTheirDesc: seenAuthors.size,
  };
};
```
