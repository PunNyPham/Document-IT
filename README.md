# Document-IT
using System;
using System.Collections.Generic;
using System.IO;

namespace Lab_DSA
{
    internal class Program
    {
        const int INF = 9999;

        public static int[,] ReadFile(string path)
        {
            if (!File.Exists(path))
            {
                Console.WriteLine("Lỗi: Không tìm thấy file!");
                return null;
            }

            string[] lines = File.ReadAllLines(path);
            if (lines.Length == 0) return null;

            int rows = lines.Length;
            string[] firstLineElements = lines[0].Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);
            int cols = firstLineElements.Length;

            int[,] matrix = new int[rows, cols];

            for (int i = 0; i < rows; i++)
            {
                string[] numbers = lines[i].Split(new[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);
                for (int j = 0; j < cols; j++)
                {
                    if (int.TryParse(numbers[j], out int value))
                        matrix[i, j] = value;
                    else
                        matrix[i, j] = INF; // Nếu không phải số, coi như không có cạnh
                }
            }
            return matrix;
        }

        public static void PrintMatrix(int[,] matrix)
        {
            if (matrix == null) return;
            int rows = matrix.GetLength(0);
            int cols = matrix.GetLength(1);
            for (int i = 0; i < rows; i++)
            {
                for (int j = 0; j < cols; j++)
                {
                    Console.Write((matrix[i, j] == INF ? "INF" : matrix[i, j].ToString()) + "\t");
                }
                Console.WriteLine();
            }
        }

        // --- Duyệt đồ thị ---

        static void BFS(int start, int size, int[,] matrix)
        {
            bool[] visited = new bool[size];
            Queue<int> queue = new Queue<int>();

            visited[start] = true;
            queue.Enqueue(start);

            Console.Write("BFS: ");
            while (queue.Count > 0)
            {
                int node = queue.Dequeue();
                Console.Write(node + " ");

                for (int v = 0; v < size; v++)
                {
                    // Giả định matrix[i,j] > 0 là có cạnh (cho đồ thị không trọng số hoặc trọng số dương)
                    if (matrix[node, v] != 0 && matrix[node, v] != INF && !visited[v])
                    {
                        visited[v] = true;
                        queue.Enqueue(v);
                    }
                }
            }
            Console.WriteLine();
        }

        static void DFS(int start, int size, int[,] matrix)
        {
            bool[] visited = new bool[size];
            Stack<int> stack = new Stack<int>();

            stack.Push(start);

            Console.Write("DFS: ");
            while (stack.Count > 0)
            {
                int node = stack.Pop();

                if (!visited[node])
                {
                    Console.Write(node + " ");
                    visited[node] = true;
                }

                // Duyệt ngược từ size-1 về 0 để khi pop ra sẽ ưu tiên đỉnh nhỏ trước (giống đệ quy)
                for (int v = size - 1; v >= 0; v--)
                {
                    if (matrix[node, v] != 0 && matrix[node, v] != INF && !visited[v])
                    {
                        stack.Push(v);
                    }
                }
            }
            Console.WriteLine();
        }

        // --- Thuật toán Kruskal ---

        static int[,] MatrixToEdges(int[,] matrix, int n, out int edgeCount)
        {
            edgeCount = 0;
            for (int i = 0; i < n; i++)
                for (int j = i + 1; j < n; j++)
                    if (matrix[i, j] != 0 && matrix[i, j] != INF) edgeCount++;

            int[,] edges = new int[edgeCount, 3];
            int index = 0;
            for (int i = 0; i < n; i++)
            {
                for (int j = i + 1; j < n; j++)
                {
                    if (matrix[i, j] != 0 && matrix[i, j] != INF)
                    {
                        edges[index, 0] = i;
                        edges[index, 1] = j;
                        edges[index, 2] = matrix[i, j];
                        index++;
                    }
                }
            }
            return edges;
        }

        static int Find(int x, int[] parent)
        {
            if (parent[x] == x) return x;
            return parent[x] = Find(parent[x], parent);
        }

        static void Kruskal(int[,] matrix, int n)
        {
            int m;
            int[,] edges = MatrixToEdges(matrix, n, out m);

            // Sắp xếp cạnh bằng thuật toán đơn giản (Sử dụng List.Sort sẽ chuyên nghiệp hơn)
            for (int i = 0; i < m - 1; i++)
                for (int j = i + 1; j < m; j++)
                    if (edges[i, 2] > edges[j, 2])
                        for (int k = 0; k < 3; k++)
                        {
                            int temp = edges[i, k];
                            edges[i, k] = edges[j, k];
                            edges[j, k] = temp;
                        }

            int[] parent = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;

            int count = 0, totalWeight = 0;
            Console.WriteLine("\nKruskal MST:");
            for (int i = 0; i < m && count < n - 1; i++)
            {
                int u = edges[i, 0];
                int v = edges[i, 1];
                if (Find(u, parent) != Find(v, parent))
                {
                    parent[Find(u, parent)] = Find(v, parent); // Union đơn giản
                    Console.WriteLine($"{u} - {v} \t Trọng số: {edges[i, 2]}");
                    totalWeight += edges[i, 2];
                    count++;
                }
            }
            Console.WriteLine("Tổng trọng số (Kruskal): " + totalWeight);
        }

        // --- Thuật toán Prim ---

        static void Prim(int[,] matrix, int size)
        {
            bool[] used = new bool[size];
            int[] dist = new int[size];
            int[] parent = new int[size];

            for (int i = 0; i < size; i++)
            {
                used[i] = false;
                dist[i] = INF;
                parent[i] = -1;
            }

            dist[0] = 0;

            for (int i = 0; i < size; i++)
            {
                int u = FindMinVertex(size, used, dist);
                if (u == -1) break;

                used[u] = true;

                for (int v = 0; v < size; v++)
                {
                    if (!used[v] && matrix[u, v] != 0 && matrix[u, v] < dist[v])
                    {
                        dist[v] = matrix[u, v];
                        parent[v] = u;
                    }
                }
            }
            PrintPrimMST(size, parent, dist);
        }
         
        static int FindMinVertex(int size, bool[] used, int[] dist)
        {
            int minDist = INF, minVertex = -1;
            for (int i = 0; i < size; i++)
            {
                if (!used[i] && dist[i] < minDist)
                {
                    minDist = dist[i];
                    minVertex = i;
                }
            }
            return minVertex;
        }

        static void PrintPrimMST(int size, int[] parent, int[] dist)
        {
            Console.WriteLine("\nPrim MST:");
            int totalWeight = 0;
            for (int i = 1; i < size; i++)
            {
                if (parent[i] != -1)
                {
                    Console.WriteLine($"{parent[i]} - {i} \t Trọng số: {dist[i]}");
                    totalWeight += dist[i];
                }
            }
            Console.WriteLine("Tổng trọng số (Prim): " + totalWeight);
        }
                public static int IsSafe(int v, int pos, int[,] matrix, int[] path)
        {
            if (matrix[path[pos - 1], v] == 0)
                return 0;

            for (int i = 0; i < pos; i++)
                if (path[i] == v)
                    return 0;

            return 1;
        }

        public static int HamiltonianUtil(int pos, int n, int[,] matrix, int[] path)
        {
            if (pos == n)
            {
                return matrix[path[n - 1], path[0]] == 1 ? 1 : 0;
            }

            for (int v = 1; v < n; v++)
            {
                if (IsSafe(v, pos, matrix, path) == 1)
                {
                    path[pos] = v;

                    if (HamiltonianUtil(pos + 1, n, matrix, path) == 1)
                        return 1;

                    path[pos] = -1;
                }
            }

            return 0;
        }


        // Hàm chính tìm chu trình Hamilton
        public static void Hamiltonian(int[,] matrix)
        {
            int n = matrix.GetLength(0);   // số đỉnh
            int[] path = new int[n];

            for (int i = 0; i < n; i++)
                path[i] = -1;

            path[0] = 0; // cố định đỉnh đầu

            if (HamiltonianUtil(1, n, matrix, path) == 0)
            {
                Console.WriteLine("Khong ton tai chu trinh Hamilton!");
                return;
            }

            Console.Write("Chu trinh Hamilton: ");
            for (int i = 0; i < n; i++)
                Console.Write(path[i] + " ");
            Console.WriteLine(path[0]);   // quay lại đỉnh đầu
        }
        static void RemoveEdge(int[,] matrix, int u, int v)
        {
            matrix[u, v] = 0;
            matrix[v, u] = 0;
        }

        // 4. Hàm DFS đếm số đỉnh liên thông
        static int DFSCount(int[,] matrix, int v, bool[] visited)
        {
            int V = matrix.GetLength(0);
            visited[v] = true;
            int count = 1;

            for (int i = 0; i < V; i++)
            {
                if (matrix[v, i] == 1 && !visited[i])
                    count += DFSCount(matrix, i, visited);
            }
            return count;
        }

        // 5. Kiểm tra cạnh (u,v) có hợp lệ để đi tiếp không
        static bool IsValidNextEdge(int[,] matrix, int u, int v)
        {
            int V = matrix.GetLength(0);
            int count = 0;
            for (int i = 0; i < V; i++)
            {
                if (matrix[u, i] == 1) count++;
            }

            // Nếu là cạnh duy nhất, phải đi nó
            if (count == 1) return true;

            // Kiểm tra nếu là cạnh cầu (bridge)
            bool[] visited = new bool[V];
            int count1 = DFSCount(matrix, u, visited);

            RemoveEdge(matrix, u, v);
            visited = new bool[V];
            int count2 = DFSCount(matrix, u, visited);

            // Khôi phục cạnh sau khi kiểm tra
            matrix[u, v] = matrix[v, u] = 1;

            return count1 <= count2;
        }

        // 6. Hàm đệ quy in chu trình/đường đi
        static void PrintEulerUtil(int[,] matrix, int u)
        {
            int V = matrix.GetLength(0);
            for (int v = 0; v < V; v++)
            {
                if (matrix[u, v] == 1 && IsValidNextEdge(matrix, u, v))
                {
                    Console.Write($"{u} --> {v}  ");
                    RemoveEdge(matrix, u, v);
                    PrintEulerUtil(matrix, v);
                }
            }
        }

        // 7. Hàm tìm đỉnh bắt đầu và thực thi
        static void PrintEulerTour(int[,] matrix)
        {
            int start = 0;
            int V = matrix.GetLength(0);
            
            [cite_start]// Tìm đỉnh bậc lẻ đầu tiên (nếu có) để xét đường đi Euler [cite: 11, 13]
            [cite_start]// Nếu không có đỉnh lẻ, chọn đỉnh đầu tiên có bậc > 0 [cite: 61]
            for (int i = 0; i < V; i++)
            {
                int deg = 0;
                for (int j = 0; j < V; j++) deg += matrix[i, j];
                if (deg % 2 != 0)
                {
                    start = i;
                    break;
                }
            }

            Console.WriteLine("\nKet qua duyet Euler:");
            PrintEulerUtil(matrix, start);
            Console.WriteLine();
        }
        static void Main(string[] args)
        {
            // Thay đổi đường dẫn file của bạn ở đây
            string path = @"D:\Tài Liệu\cấu trúc dữ liệu và giải thuật\DSA lab\Lab_DSA\Matrix.txt";
            int[,] matrix = ReadFile(path);

            if (matrix != null)
            {
                int n = matrix.GetLength(0);
                PrintMatrix(matrix);
                Console.WriteLine();

                BFS(0, n, matrix);
                DFS(0, n, matrix);
                Kruskal(matrix, n);
                Prim(matrix, n);
            }
            Console.ReadLine();
        }
    }
}
