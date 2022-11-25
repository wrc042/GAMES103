# Note

## FVM

$D_m=[X_{10}\ X_{20}\ X_{30}]$

$F=[x_{10}\ x_{20}\ x_{30}]D_m^{-1}$

$G=\frac{1}{2}(F^TF-I)$

$P=F\frac{\partial W}{\partial G}$

StVK:

$S=\frac{\partial W}{\partial G}=2s_1G+s_0tr(G)I$

$[f_1\ f_2\ f_3]=-\frac{1}{6\det{}D_m^{-1}}PD_m^{-T}$

$f_0=-f_1-f_2-f_3$


## Code

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System;
using System.IO;

public class FVM : MonoBehaviour
{
    float dt = 0.003f;
    float mass = 1;
    float stiffness_0 = 20000.0f;
    float stiffness_1 = 5000.0f;
    float damp = 0.999f;

    int[] Tet;
    int tet_number;         //The number of tetrahedra

    Vector3[] Force;
    Vector3[] V;
    Vector3[] X;
    int number;             //The number of vertices

    Matrix4x4[] inv_Dm;

    //For Laplacian smoothing.
    Vector3[] V_sum;
    int[] V_num;

    SVD svd = new SVD();
    Vector3 g = new Vector3(0, -9.8f, 0);
    Vector3 floor_normal = new Vector3(0, 1, 0);
    float floor_y = -3f;
    HashSet<int> edge_set = new HashSet<int>();
    float lambda_smooth = 0.1f;

    // Start is called before the first frame update
    void Start()
    {
        // FILO IO: Read the house model from files.
        // The model is from Jonathan Schewchuk's Stellar lib.
        {
            string fileContent = File.ReadAllText("Assets/house2.ele");
            string[] Strings = fileContent.Split(new char[] { ' ', '\t', '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);

            tet_number = int.Parse(Strings[0]);
            Tet = new int[tet_number * 4];

            for (int tet = 0; tet < tet_number; tet++)
            {
                Tet[tet * 4 + 0] = int.Parse(Strings[tet * 5 + 4]) - 1;
                Tet[tet * 4 + 1] = int.Parse(Strings[tet * 5 + 5]) - 1;
                Tet[tet * 4 + 2] = int.Parse(Strings[tet * 5 + 6]) - 1;
                Tet[tet * 4 + 3] = int.Parse(Strings[tet * 5 + 7]) - 1;
            }
        }
        {
            string fileContent = File.ReadAllText("Assets/house2.node");
            string[] Strings = fileContent.Split(new char[] { ' ', '\t', '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);
            number = int.Parse(Strings[0]);
            X = new Vector3[number];
            for (int i = 0; i < number; i++)
            {
                X[i].x = float.Parse(Strings[i * 5 + 5]) * 0.4f;
                X[i].y = float.Parse(Strings[i * 5 + 6]) * 0.4f;
                X[i].z = float.Parse(Strings[i * 5 + 7]) * 0.4f;
            }
            //Centralize the model.
            Vector3 center = Vector3.zero;
            for (int i = 0; i < number; i++) center += X[i];
            center = center / number;
            for (int i = 0; i < number; i++)
            {
                X[i] -= center;
                float temp = X[i].y;
                X[i].y = X[i].z;
                X[i].z = temp;
            }
        }
        /*tet_number=1;
        Tet = new int[tet_number*4];
        Tet[0]=0;
        Tet[1]=1;
        Tet[2]=2;
        Tet[3]=3;

        number=4;
        X = new Vector3[number];
        V = new Vector3[number];
        Force = new Vector3[number];
        X[0]= new Vector3(0, 0, 0);
        X[1]= new Vector3(1, 0, 0);
        X[2]= new Vector3(0, 1, 0);
        X[3]= new Vector3(0, 0, 1);*/


        //Create triangle mesh.
        Vector3[] vertices = new Vector3[tet_number * 12];
        int vertex_number = 0;
        for (int tet = 0; tet < tet_number; tet++)
        {
            vertices[vertex_number++] = X[Tet[tet * 4 + 0]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 2]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 1]];

            vertices[vertex_number++] = X[Tet[tet * 4 + 0]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 3]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 2]];

            vertices[vertex_number++] = X[Tet[tet * 4 + 0]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 1]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 3]];

            vertices[vertex_number++] = X[Tet[tet * 4 + 1]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 2]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 3]];
        }

        int[] triangles = new int[tet_number * 12];
        for (int t = 0; t < tet_number * 4; t++)
        {
            triangles[t * 3 + 0] = t * 3 + 0;
            triangles[t * 3 + 1] = t * 3 + 1;
            triangles[t * 3 + 2] = t * 3 + 2;
        }
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        mesh.vertices = vertices;
        mesh.triangles = triangles;
        mesh.RecalculateNormals();


        V = new Vector3[number];
        Force = new Vector3[number];
        V_sum = new Vector3[number];
        V_num = new int[number];

        //TODO: Need to allocate and assign inv_Dm
        inv_Dm = new Matrix4x4[tet_number];
        for (int tet = 0; tet < tet_number; tet++)
        {
            inv_Dm[tet] = Build_Edge_Matrix(tet).inverse;
        }
    }

    Matrix4x4 Build_Edge_Matrix(int tet)
    {
        Matrix4x4 ret = Matrix4x4.zero;
        //TODO: Need to build edge matrix here.
        Vector3 X0 = X[Tet[tet * 4 + 0]];
        Vector3 X10 = X[Tet[tet * 4 + 1]] - X0;
        Vector3 X20 = X[Tet[tet * 4 + 2]] - X0;
        Vector3 X30 = X[Tet[tet * 4 + 3]] - X0;
        ret.SetColumn(0, new Vector4(X10.x, X10.y, X10.z, 0));
        ret.SetColumn(1, new Vector4(X20.x, X20.y, X20.z, 0));
        ret.SetColumn(2, new Vector4(X30.x, X30.y, X30.z, 0));
        ret.SetColumn(3, new Vector4(0, 0, 0, 1));

        return ret;
    }

    bool Edge_Counted(int x, int y)
    {
        return false;
        // int idx = Math.Min(x, y) * number + Math.Max(x, y);
        // if (edge_set.Contains(idx))
        // {
        //     return true;
        // }
        // else
        // {
        //     edge_set.Add(idx);
        //     return false;
        // }
    }

    void Add_Edge(int x, int y)
    {
        if (!Edge_Counted(x, y))
        {
            V_sum[x] += V[y];
            V_sum[y] += V[x];
            V_num[y] += 1;
            V_num[x] += 1;
        }
    }

    void Smooth_V()
    {
        edge_set = new HashSet<int>();
        for (int i = 0; i < number; i++)
        {
            V_num[i] = 0;
            V_sum[i] = Vector3.zero;
        }

        for (int tet = 0; tet < tet_number; tet++)
        {
            int idx0 = Tet[tet * 4 + 0];
            int idx1 = Tet[tet * 4 + 1];
            int idx2 = Tet[tet * 4 + 2];
            int idx3 = Tet[tet * 4 + 3];

            Add_Edge(idx0, idx1);
            Add_Edge(idx0, idx2);
            Add_Edge(idx0, idx3);
            Add_Edge(idx1, idx2);
            Add_Edge(idx1, idx3);
            Add_Edge(idx2, idx3);
        }

        for (int i = 0; i < number; i++)
        {
            V[i] = (1.0f - lambda_smooth) * V[i] + lambda_smooth * (V_sum[i] / V_num[i]);
        }
    }


    void _Update()
    {
        // Jump up.
        if (Input.GetKeyDown(KeyCode.Space))
        {
            for (int i = 0; i < number; i++)
                V[i].y += 0.2f;
        }

        for (int i = 0; i < number; i++)
        {
            //TODO: Add gravity to Force.
            Force[i] = mass * g;
        }

        for (int tet = 0; tet < tet_number; tet++)
        {
            //TODO: Deformation Gradient
            Matrix4x4 F = Build_Edge_Matrix(tet) * inv_Dm[tet];

            //TODO: Green Strain
            Matrix4x4 G = F.transpose * F;

            for (int i = 0; i < 4; i++)
                for (int j = 0; j < 4; j++)
                    G[i, j] = (G[i, j] - Matrix4x4.identity[i, j]) / 2;

            //TODO: Second PK Stress
            float G_trace = G[0, 0] + G[1, 1] + G[2, 2];
            Matrix4x4 S = Matrix4x4.identity;
            for (int i = 0; i < 4; i++)
            {
                for (int j = 0; j < 4; j++)
                {
                    S[i, j] = 2 * stiffness_1 * G[i, j] + stiffness_0 * G_trace * Matrix4x4.identity[i, j];
                }
            }

            //TODO: Elastic Force
            Matrix4x4 force = F * S * inv_Dm[tet].transpose;
            float vol = 6 * inv_Dm[tet].determinant;
            for (int i = 0; i < 4; i++)
                for (int j = 0; j < 4; j++)
                    force[i, j] *= -1f / vol;
            Vector3 f1 = force.GetColumn(0);
            Vector3 f2 = force.GetColumn(1);
            Vector3 f3 = force.GetColumn(2);
            Force[Tet[tet * 4 + 1]] += f1;
            Force[Tet[tet * 4 + 2]] += f2;
            Force[Tet[tet * 4 + 3]] += f3;
            Force[Tet[tet * 4 + 0]] += -f1 - f2 - f3;
        }

        Smooth_V();

        for (int i = 0; i < number; i++)
        {
            //TODO: Update X and V here.
            V[i] += dt * Force[i] / mass;
            X[i] += V[i] * dt;
            V[i] *= damp;

            //TODO: (Particle) collision with floor.
            // no friction
            if (X[i].y < floor_y)
            {
                V[i].y += (floor_y - X[i].y) / dt;
                X[i].y = floor_y;
            }
        }
    }

    // Update is called once per frame
    void Update()
    {
        for (int l = 0; l < 10; l++)
            _Update();

        // Dump the vertex array for rendering.
        Vector3[] vertices = new Vector3[tet_number * 12];
        int vertex_number = 0;
        for (int tet = 0; tet < tet_number; tet++)
        {
            vertices[vertex_number++] = X[Tet[tet * 4 + 0]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 2]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 1]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 0]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 3]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 2]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 0]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 1]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 3]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 1]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 2]];
            vertices[vertex_number++] = X[Tet[tet * 4 + 3]];
        }
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        mesh.vertices = vertices;
        mesh.RecalculateNormals();
    }
}
```