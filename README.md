### 2D to 3D Map Unity 

### Code

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class LevelCreator : MonoBehaviour
{

    Texture2D imageToRead; // set image being readable
    private GameObject[] imageWall;
 
    Sprite[] pictures; 

    public Color deleteColor; // if we see transparent color cut it out from output
    public Mesh FlatCube;    
 
    public GameObject cube;
    public GameObject parent;

    public Material borderMat; 

    int numberOfCubes;
    int nCubes;

    public Material cubeMat;

    MaterialPropertyBlock _propBlock;

    [System.Serializable]
    public class Pool {

        public string tag;  // add a tag to your go
        public GameObject prefab; // the object you want to spawn for the map
        public int size; // size of image your reading so 10x10 = 100
    }

    public List<Pool> pools;
    ManagerNew manager;

    public Dictionary<string, Queue<GameObject>> poolDictionary;

    void Start()
    {
        manager = FindObjectOfType<ManagerNew>();
        pictures = manager.GetSprites();
        poolDictionary = new Dictionary<string, Queue<GameObject>>();

        _propBlock = new MaterialPropertyBlock();

        foreach(Pool pool in pools)
        {
            Queue<GameObject> objectPool = new Queue<GameObject>();

            for(int i = 0; i < pool.size; i++)
            {
                GameObject obj = Instantiate(pool.prefab);
                obj.SetActive(false);
                objectPool.Enqueue(obj);

            }

            poolDictionary.Add(pool.tag, objectPool);

        }
    }

    public void SpawnFromPool(string tag, Vector3 position, Color color, bool border)
    {

        if (!poolDictionary.ContainsKey(tag))
        {

            Debug.Log("Pool doesnt exist");
            return;
        }

        GameObject objectToSpawn = poolDictionary[tag].Dequeue();


        objectToSpawn.SetActive(true);
        objectToSpawn.transform.position = position;

        objectToSpawn.transform.SetParent(parent.transform);

        if (!border)
        {


            objectToSpawn.GetComponent<MeshRenderer>().GetPropertyBlock(_propBlock);

            _propBlock.SetColor("_Color", color);
            objectToSpawn.GetComponent<MeshRenderer>().SetPropertyBlock(_propBlock);

             objectToSpawn.GetComponent<MeshRenderer>().enabled = false;

            
        } else
        {
            objectToSpawn.GetComponent<MeshFilter>().mesh = FlatCube;
            objectToSpawn.GetComponent<MeshRenderer>().material = borderMat;
            objectToSpawn.tag = "Border";
            Destroy(objectToSpawn.GetComponent<AudioSource>());
            Destroy(objectToSpawn.GetComponent<BoxCollider>());
            Destroy(objectToSpawn.GetComponent<Cube>());
        }
    
        poolDictionary[tag].Enqueue(objectToSpawn);
    }

    public int GetCubes()
    {
        return numberOfCubes;
    }

    public void CreateLevel(string name)
    {
        // Debug.Log("Searching For: " + name);

        for (int i = 0; i < pictures.Length; i++)
        {
            if (pictures[i].name == name)
            {
                //Debug.Log("size trec: " + (int)pictures[i].textureRect.width + " " + (int)pictures[i].textureRect.height);
               // Debug.Log("size trec: " + (int)pictures[i].rect.width + " " + (int)pictures[i].rect.height);

                Texture2D croppedTexture = new Texture2D((int)pictures[i].rect.width, (int)pictures[i].rect.height);
                var pixels = pictures[i].texture.GetPixels((int)pictures[i].rect.x,
                                                        (int)pictures[i].rect.y,
                                                        (int)pictures[i].rect.width,
                                                        (int)pictures[i].rect.height);
                croppedTexture.SetPixels(pixels);
                croppedTexture.Apply();

                imageToRead = croppedTexture;
                createImage();

                break;
            }
        }
    }

    void createImage() {

        Color[] pixels = imageToRead.GetPixels();
       
        imageWall = new GameObject[pixels.Length];

        for(int i = 0; i < pixels.Length; i++)
        {
            //border
            if(pixels[i] == deleteColor)
            {
                SpawnFromPool("Cube", new Vector3(i % imageToRead.width, 0.01f, i / imageToRead.width), pixels[i], true);
                numberOfCubes -= 1;

            }
            else
            {
                SpawnFromPool("Cube", new Vector3(i % imageToRead.width, 1.4f, i / imageToRead.width), pixels[i], false);
                numberOfCubes += 1;

            }

        }

    }

}
```
