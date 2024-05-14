# OBJ Model Importing in Real-Time with Textures in Unity: A Step-by-Step Guide

Are you attempting to dynamically import 3D models into your Unity project without the need to download them onto your device?

Look no further! This article provides you with a step-by-step guide on how to import OBJ models in real-time using a URL directly to the model. Unlock the solution to seamlessly integrating models into your project without the hassle of storing them locally.

![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/245a2ec5-0c66-4593-a7c7-018ffc99f388)


# Prerequisite Things
Latest version Unity

URL for an obj model

# What is OBJ?
OBJ (object) file consists of a 3D model written as a code.

An OBJ file is downloaded with it's MTL (material) file, the mtl file is liked with the obj file by a line in obj as: mtllib <MTL file reference>

[Sample](https://github.com/DamanAhuja/cube/blob/main/1/cube.obj) OBJ File 

## MTL File
An MTL File is a code file to define the materials used in the model.

It consists of:

Ka (Ambient color)

Kd (Diffuse color)

Ks (Specular color)

Ns (Specular Exponent)

Tr (Transparency)

Ni (Optical Density)

illum (Illumination Model)

The texture is mapped to the material by a command: map_kd <texture reference>

[Sample](https://github.com/DamanAhuja/cube/blob/main/1/cube.mtl)  MTL FIle 

![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/1aac268c-33fa-427e-ae0e-664b15d29a67)

# Importing The Model in Real Time

Let's get started with our unity project.

First of all, create a new project in unity hub. 

![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/c3eee9ce-927b-4afd-a3ca-733eb8512527)


Then we have to import a unity package named "Runtime OBJ Importer".

[Here](https://assetstore.unity.com/packages/tools/modeling/runtime-obj-importer-49547) is a link to get it.  

After Importing this package you will have to different sample scenes in your Project. One is for importing by Path, which takes the file path as input and the other one as "import from stream" which imports a model from the given OBJ url.


![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/f2a5d1b2-fdc3-4655-8b77-44d90462b96a)

Now if you have the OBJ, MTL, Texture files downloaded in your Computer then you can simply import it in Run-Time by just entering the file path in the "Import by Path" scene.

But now as we try to import the model from stream, it will get the model imported but you will see that there will be no texture applied to the model. For getting the model with textures we will have to make some changes in the scripts present in the package.

## OBJLoader Script
You will find an OBJLoader Script in your assets. Here I will tell you some changes to make in this script.
Search for the Method named "LoadMaterialLibrary()" in the script and then change the whole method by -
```
public void LoadMaterialLibrary(string mtlLibPath)
{
    if (_objInfo != null)
    {
        // Construct path relative to OBJ model directory
        string relativePath = Path.Combine(_objInfo.Directory.FullName, mtlLibPath);
        Debug.Log("Checking MTL file at relative path: " + relativePath);

        if (File.Exists(relativePath))
        {
            Materials = new MTLLoader().Load(relativePath);
            return;
        }
    }
    if (Uri.TryCreate(mtlLibPath, UriKind.Absolute, out Uri uriResult) && (uriResult.Scheme == Uri.UriSchemeHttp || uriResult.Scheme == Uri.UriSchemeHttps))
    {
        Debug.Log("Loading MTL file from URL: " + mtlLibPath);
        Materials = new MTLLoader().LoadFromURL(mtlLibPath);
        return;
    }

    // Fallback: Check provided path directly
    Debug.Log("Checking MTL file at provided path: " + mtlLibPath);
    if (File.Exists(mtlLibPath))
    {
        Materials = new MTLLoader().Load(mtlLibPath);
        return;
    }

    // Consider adding error handling for missing MTL file
    Debug.LogError("MTL file not found: " + mtlLibPath);
```


![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/6bb6a994-3a74-4965-a71f-b9d33fd438d0)


Let me now explain the changes we just made.

First of all, we are adding script to check if the content after "mtllib" is an URL or a local path.

And if it is a URL then it calls a function from the MTLLoader file named LoadFromURL().

Now we will define the LoadFromURL() in the MTLLoader script

## MTLLoader
Find the MTLLoader script in your assets and then add this code snippet in the already existing code.
```
public Dictionary<string, Material> LoadFromURL(string url)
    {
        try
        {
            WebClient client = new WebClient();
            string mtlData = client.DownloadString(url);
            using (var stream = new MemoryStream(System.Text.Encoding.UTF8.GetBytes(mtlData)))
            {
                return Load(stream);
            }
        }
        catch (Exception e)
        {
            Debug.LogError("Error loading MTL file from URL: " + e.Message);
            return null;
        }
    }
```
Now as you see, what this function does is reading the content from URL and making a text stream from its data. Then it calls the already existing Load() function which takes argument as a text stream.

Then find the Load() function in that script only, and then find the script -
```
//diffuse map
            if (splitLine[0] == "map_Kd" || splitLine[0] == "map_kd")
            {
                string texturePath = GetTexPathFromMapStatement(processedLine, splitLine);
                Debug.Log("loading texture from: " + texturePath);
                if(texturePath == null)
                {
                    continue; //invalid args or sth
                }
```
Just after this script add an if condition -
```
if (Uri.TryCreate(texturePath, UriKind.Absolute, out Uri uriResult) && (uriResult.Scheme == Uri.UriSchemeHttp || uriResult.Scheme == Uri.UriSchemeHttps))
                {
                    Texture2D texture = LoadTextureFromURL(texturePath);
                    Debug.Log(texture);
                    if (texture != null)
                    {
                        currentMaterial.SetTexture("_MainTex",texture);
                    }
                }
```
This condition checks if the reference to the texture is given as URL, and if it is true, a function named LoadTextureFromURL() is called.

![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/01896893-d813-4683-92db-25809051b3fb)

## Defining LoadTextureFromURL()

Now in the last step we will just define the required function which will load the texture by taking the argument as an URL.
```
public Texture2D LoadTextureFromURL(string url)
    {
        try
        {
            WebClient client = new WebClient();
            byte[] textureData = client.DownloadData(url);
            Texture2D texture = new Texture2D(2, 2);
            if (texture.LoadImage(textureData))
            {
                return texture;
            }
            else
            {
                Debug.LogError("Failed to load texture from URL: " + url);
                return null;
            }
        }
        catch (Exception e)
        {
            Debug.LogError("Error loading texture from URL: " + e.Message);
            return null;
        }
    }
```
## Testing -
Now you are all done with changes to be made.

To try your newly made project, just open the LoadFromStream script from the assets and change the input URL from the default to -
https://raw.githubusercontent.com/DamanAhuja/cube/main/1/cube.obj

![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/2fd1af2a-86ec-4a08-9fa8-1bbbcb3aff28)

Just run your scene and you will find a OBJ model imported in your scene.

# Some Additional Tips:
Now you might be having issues with the textures of the model imported, the model might not be looking good.

This issue is happening because of the shader of the imported model.
We can change the shader back to standard after importing the model. To change it, we have to add a function in the LoadFromStream script.
```
private void ChangeShaderToStandard(GameObject obj)
{
    if (obj != null)
    {
        Renderer[] renderers = obj.GetComponentsInChildren<Renderer>();
        foreach (Renderer renderer in renderers)
        {
            Material[] materials = renderer.materials;
            foreach (Material material in materials)
            {
                material.shader = Shader.Find("Standard");
            }
        }
    }
}
```
Now call this function in Start() function by -
```
ChangeShaderToStandard(loadedobj);
```


![image](https://github.com/DamanAhuja/OBJLoader/assets/142963733/bec59e5c-a4c6-4d52-b3f0-bfe8b6f5bd99)

Now you will get the model with proper textures.
