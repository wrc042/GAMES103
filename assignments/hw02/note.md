# Note

## Implicit

$y=x+hv$

$x'=x-(\frac{1}{h^2}+4k)^{-1}g$

$x'=\argmin_x\frac{1}{2\Delta t^2}|x-y|_M^2+E(x)$

$g=\frac{1}{h^2}M(x-y)-f(x)$

$v=v+\frac{1}{h}(x-y)$

### Chebyshev Acceleration

```python
if (k == 0):
    w = 1
else if (k == 1):
    w = 2/(2 - rho ** 2)
else:
    w = 4/(4 - w * rho ** 2)

old_x = x
x = x + alpha * D.inv() * r
x = x * w + (1 - w) * last_x
last_x = old_x
```

### Collision Handling

$v=v+\frac{1}{h}(c+r\frac{x-c}{||x-c||}-x)$

$x=c+r\frac{x-c}{||x-c||}$

## PBD

Strain limiting

## Code

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class implicit_model : MonoBehaviour
{
    float t = 0.0333f;
    float mass = 1;
    float damping = 0.99f;
    float rho = 0.995f;
    float spring_k = 8000;
    int[] E;
    float[] L;
    Vector3[] V;
    Vector3 g = new Vector3(0, -9.8f, 0);
    int n_vertice = 21;
    float radius = 2.7f;

    // Start is called before the first frame update
    void Start()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;

        //Resize the mesh.
        int n = n_vertice;
        Vector3[] X = new Vector3[n * n];
        Vector2[] UV = new Vector2[n * n];
        int[] triangles = new int[(n - 1) * (n - 1) * 6];
        for (int j = 0; j < n; j++)
            for (int i = 0; i < n; i++)
            {
                X[j * n + i] = new Vector3(5 - 10.0f * i / (n - 1), 0, 5 - 10.0f * j / (n - 1));
                UV[j * n + i] = new Vector3(i / (n - 1.0f), j / (n - 1.0f));
            }
        int t = 0;
        for (int j = 0; j < n - 1; j++)
            for (int i = 0; i < n - 1; i++)
            {
                triangles[t * 6 + 0] = j * n + i;
                triangles[t * 6 + 1] = j * n + i + 1;
                triangles[t * 6 + 2] = (j + 1) * n + i + 1;
                triangles[t * 6 + 3] = j * n + i;
                triangles[t * 6 + 4] = (j + 1) * n + i + 1;
                triangles[t * 6 + 5] = (j + 1) * n + i;
                t++;
            }
        mesh.vertices = X;
        mesh.triangles = triangles;
        mesh.uv = UV;
        mesh.RecalculateNormals();


        //Construct the original E
        int[] _E = new int[triangles.Length * 2];
        for (int i = 0; i < triangles.Length; i += 3)
        {
            _E[i * 2 + 0] = triangles[i + 0];
            _E[i * 2 + 1] = triangles[i + 1];
            _E[i * 2 + 2] = triangles[i + 1];
            _E[i * 2 + 3] = triangles[i + 2];
            _E[i * 2 + 4] = triangles[i + 2];
            _E[i * 2 + 5] = triangles[i + 0];
        }
        //Reorder the original edge list
        for (int i = 0; i < _E.Length; i += 2)
            if (_E[i] > _E[i + 1])
                Swap(ref _E[i], ref _E[i + 1]);
        //Sort the original edge list using quicksort
        Quick_Sort(ref _E, 0, _E.Length / 2 - 1);

        int e_number = 0;
        for (int i = 0; i < _E.Length; i += 2)
            if (i == 0 || _E[i + 0] != _E[i - 2] || _E[i + 1] != _E[i - 1])
                e_number++;

        E = new int[e_number * 2];
        for (int i = 0, e = 0; i < _E.Length; i += 2)
            if (i == 0 || _E[i + 0] != _E[i - 2] || _E[i + 1] != _E[i - 1])
            {
                E[e * 2 + 0] = _E[i + 0];
                E[e * 2 + 1] = _E[i + 1];
                e++;
            }

        L = new float[E.Length / 2];
        for (int e = 0; e < E.Length / 2; e++)
        {
            int v0 = E[e * 2 + 0];
            int v1 = E[e * 2 + 1];
            L[e] = (X[v0] - X[v1]).magnitude;
        }

        V = new Vector3[X.Length];
        for (int i = 0; i < V.Length; i++)
            V[i] = new Vector3(0, 0, 0);
    }

    void Quick_Sort(ref int[] a, int l, int r)
    {
        int j;
        if (l < r)
        {
            j = Quick_Sort_Partition(ref a, l, r);
            Quick_Sort(ref a, l, j - 1);
            Quick_Sort(ref a, j + 1, r);
        }
    }

    int Quick_Sort_Partition(ref int[] a, int l, int r)
    {
        int pivot_0, pivot_1, i, j;
        pivot_0 = a[l * 2 + 0];
        pivot_1 = a[l * 2 + 1];
        i = l;
        j = r + 1;
        while (true)
        {
            do ++i; while (i <= r && (a[i * 2] < pivot_0 || a[i * 2] == pivot_0 && a[i * 2 + 1] <= pivot_1));
            do --j; while (a[j * 2] > pivot_0 || a[j * 2] == pivot_0 && a[j * 2 + 1] > pivot_1);
            if (i >= j) break;
            Swap(ref a[i * 2], ref a[j * 2]);
            Swap(ref a[i * 2 + 1], ref a[j * 2 + 1]);
        }
        Swap(ref a[l * 2 + 0], ref a[j * 2 + 0]);
        Swap(ref a[l * 2 + 1], ref a[j * 2 + 1]);
        return j;
    }

    void Swap(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }

    void Collision_Handling()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        Vector3[] X = mesh.vertices;

        //Handle colllision.
        Vector3 center = GameObject.Find("Sphere").transform.position;
        for (int i = 0; i < X.Length; i++)
        {

            if (i == 0 || i == (n_vertice - 1))
                continue;
            Vector3 dx = X[i] - center;
            if (dx.magnitude < radius)
            {
                V[i] = V[i] + (radius * dx / dx.magnitude - dx) / t;
                X[i] = center + radius * dx / dx.magnitude;
            }
        }


        mesh.vertices = X;
    }

    void Get_Gradient(Vector3[] X, Vector3[] X_hat, float t, Vector3[] G)
    {
        //Momentum and Gravity.
        for (int i = 0; i < X.Length; i++)
        {
            G[i] = 1 / (t * t) * mass * (X[i] - X_hat[i]);
            G[i] -= mass * g;
        }
        //Spring Force.
        for (int i = 0; i < E.Length; i += 2)
        {
            int idx0 = E[i];
            int idx1 = E[i + 1];
            Vector3 dX = X[idx0] - X[idx1];
            Vector3 tmp = spring_k * (1 - L[i / 2] / dX.magnitude) * dX;
            G[idx0] += tmp;
            G[idx1] -= tmp;
        }

    }

    // Update is called once per frame
    void Update()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        Vector3[] X = mesh.vertices;
        Vector3[] last_X = new Vector3[X.Length];
        Vector3[] X_hat = new Vector3[X.Length];
        Vector3[] G = new Vector3[X.Length];

        //Initial Setup.
        for (int i = 0; i < X.Length; i++)
        {
            if (i == 0 || i == (n_vertice - 1))
                continue;
            V[i] = damping * V[i];
            last_X[i] = X[i];
            X_hat[i] = X[i] + t * V[i];
        }

        for (int k = 0; k < 32; k++)
        {
            Get_Gradient(X, X_hat, t, G);

            //Update X by gradient.
            float w = 1;
            if (k == 1)
                w = 2 / (2 - rho * rho);
            else if (k > 1)
                w = 4 / (4 - rho * rho * w);
            for (int i = 0; i < X.Length; i++)
            {

                if (i == 0 || i == (n_vertice - 1))
                    continue;
                // X[i] = X[i] - 1 / (mass / (t * t) + 4 * spring_k) * G[i];
                Vector3 old_X = X[i];
                X[i] = X[i] - 1 / (mass / (t * t) + 4 * spring_k) * G[i];
                X[i] = w * X[i] + (1 - w) * last_X[i];
                last_X[i] = old_X;
            }
        }

        //Finishing.
        for (int i = 0; i < X.Length; i++)
        {

            if (i == 0 || i == (n_vertice - 1))
                continue;
            V[i] += (X[i] - X_hat[i]) / t;
        }

        mesh.vertices = X;

        Collision_Handling();
        mesh.RecalculateNormals();
    }
}
```

```cs
using UnityEngine;
using System.Collections;

public class PBD_model : MonoBehaviour
{

    float t = 0.0333f;
    float damping = 0.99f;
    int[] E;
    float[] L;
    Vector3[] V;
    Vector3 g = new Vector3(0, -9.8f, 0);
    int n_vertice = 21;
    float radius = 2.7f;


    // Use this for initialization
    void Start()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;

        //Resize the mesh.
        int n = n_vertice;
        Vector3[] X = new Vector3[n * n];
        Vector2[] UV = new Vector2[n * n];
        int[] T = new int[(n - 1) * (n - 1) * 6];
        for (int j = 0; j < n; j++)
            for (int i = 0; i < n; i++)
            {
                X[j * n + i] = new Vector3(5 - 10.0f * i / (n - 1), 0, 5 - 10.0f * j / (n - 1));
                UV[j * n + i] = new Vector3(i / (n - 1.0f), j / (n - 1.0f));
            }
        int t = 0;
        for (int j = 0; j < n - 1; j++)
            for (int i = 0; i < n - 1; i++)
            {
                T[t * 6 + 0] = j * n + i;
                T[t * 6 + 1] = j * n + i + 1;
                T[t * 6 + 2] = (j + 1) * n + i + 1;
                T[t * 6 + 3] = j * n + i;
                T[t * 6 + 4] = (j + 1) * n + i + 1;
                T[t * 6 + 5] = (j + 1) * n + i;
                t++;
            }
        mesh.vertices = X;
        mesh.triangles = T;
        mesh.uv = UV;
        mesh.RecalculateNormals();

        //Construct the original edge list
        int[] _E = new int[T.Length * 2];
        for (int i = 0; i < T.Length; i += 3)
        {
            _E[i * 2 + 0] = T[i + 0];
            _E[i * 2 + 1] = T[i + 1];
            _E[i * 2 + 2] = T[i + 1];
            _E[i * 2 + 3] = T[i + 2];
            _E[i * 2 + 4] = T[i + 2];
            _E[i * 2 + 5] = T[i + 0];
        }
        //Reorder the original edge list
        for (int i = 0; i < _E.Length; i += 2)
            if (_E[i] > _E[i + 1])
                Swap(ref _E[i], ref _E[i + 1]);
        //Sort the original edge list using quicksort
        Quick_Sort(ref _E, 0, _E.Length / 2 - 1);

        int e_number = 0;
        for (int i = 0; i < _E.Length; i += 2)
            if (i == 0 || _E[i + 0] != _E[i - 2] || _E[i + 1] != _E[i - 1])
                e_number++;

        E = new int[e_number * 2];
        for (int i = 0, e = 0; i < _E.Length; i += 2)
            if (i == 0 || _E[i + 0] != _E[i - 2] || _E[i + 1] != _E[i - 1])
            {
                E[e * 2 + 0] = _E[i + 0];
                E[e * 2 + 1] = _E[i + 1];
                e++;
            }

        L = new float[E.Length / 2];
        for (int e = 0; e < E.Length / 2; e++)
        {
            int i = E[e * 2 + 0];
            int j = E[e * 2 + 1];
            L[e] = (X[i] - X[j]).magnitude;
        }

        V = new Vector3[X.Length];
        for (int i = 0; i < X.Length; i++)
            V[i] = new Vector3(0, 0, 0);
    }

    void Quick_Sort(ref int[] a, int l, int r)
    {
        int j;
        if (l < r)
        {
            j = Quick_Sort_Partition(ref a, l, r);
            Quick_Sort(ref a, l, j - 1);
            Quick_Sort(ref a, j + 1, r);
        }
    }

    int Quick_Sort_Partition(ref int[] a, int l, int r)
    {
        int pivot_0, pivot_1, i, j;
        pivot_0 = a[l * 2 + 0];
        pivot_1 = a[l * 2 + 1];
        i = l;
        j = r + 1;
        while (true)
        {
            do ++i; while (i <= r && (a[i * 2] < pivot_0 || a[i * 2] == pivot_0 && a[i * 2 + 1] <= pivot_1));
            do --j; while (a[j * 2] > pivot_0 || a[j * 2] == pivot_0 && a[j * 2 + 1] > pivot_1);
            if (i >= j) break;
            Swap(ref a[i * 2], ref a[j * 2]);
            Swap(ref a[i * 2 + 1], ref a[j * 2 + 1]);
        }
        Swap(ref a[l * 2 + 0], ref a[j * 2 + 0]);
        Swap(ref a[l * 2 + 1], ref a[j * 2 + 1]);
        return j;
    }

    void Swap(ref int a, ref int b)
    {
        int temp = a;
        a = b;
        b = temp;
    }

    void Strain_Limiting()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        Vector3[] vertices = mesh.vertices;

        //Apply PBD here.
        Vector3[] sum_x = new Vector3[vertices.Length];
        int[] sum_n = new int[vertices.Length];

        for (int i = 0; i < E.Length; i += 2)
        {
            int idx0 = E[i];
            int idx1 = E[i + 1];
            Vector3 dX = vertices[idx0] - vertices[idx1];
            Vector3 tmp = L[i / 2] * dX / dX.magnitude;
            sum_x[idx0] += (vertices[idx0] + vertices[idx1] + tmp) / 2;
            sum_x[idx1] += (vertices[idx0] + vertices[idx1] - tmp) / 2;
            sum_n[idx0] += 1;
            sum_n[idx1] += 1;
        }
        for (int i = 0; i < vertices.Length; i++)
        {
            if (i == 0 || i == (n_vertice - 1))
                continue;
            Vector3 x_tmp = (0.2f * vertices[i] + sum_x[i]) / (0.2f + sum_n[i]);
            V[i] += (x_tmp - vertices[i]) / t;
            vertices[i] = x_tmp;
        }

        mesh.vertices = vertices;
    }

    void Collision_Handling()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        Vector3[] X = mesh.vertices;

        //For every vertex, detect collision and apply impulse if needed.
        Vector3 center = GameObject.Find("Sphere").transform.position;
        for (int i = 0; i < X.Length; i++)
        {

            if (i == 0 || i == (n_vertice - 1))
                continue;
            Vector3 dx = X[i] - center;
            if (dx.magnitude < radius)
            {
                V[i] = V[i] + (radius * dx / dx.magnitude - dx) / t;
                X[i] = center + radius * dx / dx.magnitude;
            }
        }
        mesh.vertices = X;
    }

    // Update is called once per frame
    void Update()
    {
        Mesh mesh = GetComponent<MeshFilter>().mesh;
        Vector3[] X = mesh.vertices;

        for (int i = 0; i < X.Length; i++)
        {
            if (i == 0 || i == (n_vertice - 1)) continue;
            //Initial Setup
            V[i] = damping * V[i];
            V[i] += g * t;
            X[i] = X[i] + t * V[i];
        }
        mesh.vertices = X;

        for (int l = 0; l < 32; l++)
            Strain_Limiting();

        Collision_Handling();

        mesh.RecalculateNormals();

    }


}
```
