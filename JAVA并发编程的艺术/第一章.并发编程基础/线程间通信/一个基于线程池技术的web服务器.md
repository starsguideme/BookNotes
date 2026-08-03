**简易 Web 服务器的完整实现**
### 一、时序回顾
客户端                                服务器
  |                                     |
  |-- ① 发起 TCP 连接 ----------------->|
  |                                     |-- ② 解析端口号与根路径
  |                                     |-- ③ 检查线程池是否已启动
  |                                     |       ├── 未启动 → 初始化线程池
  |                                     |       └── 已启动 → 继续
  |                                     |-- ④ 将请求封装为任务，提交给线程池
  |                                     |-- ⑤ 工作者线程异步处理，返回状态码
  |<-- ⑥ 返回响应（200/403/404/500）----|

### 二、完整代码实现

j

import java.io.*;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.LinkedList;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;
// ==================== 线程池接口 ====================
interface ThreadPool {
    void execute(Job job);
    void shutdown();
}
interface Job {
    void execute();
}
// ==================== 简易线程池实现 ====================
class SimpleThreadPool implements ThreadPool {
    private final LinkedList<**`Job`**> jobs = new LinkedList<>();
    private final List<**`Worker`**> workers = new LinkedList<>();
    private static final AtomicInteger threadNum = new AtomicInteger();
    private static final int DEFAULT_WORKERS = 5;
    public SimpleThreadPool() {
        this(DEFAULT_WORKERS);
    }
    public SimpleThreadPool(int num) {
        for (int i = 0; i < num; i++) {
            Worker worker = new Worker();
            workers.add(worker);
            new Thread(worker, "Worker-" + threadNum.incrementAndGet()).start();
        }
    }
    @Override
    public void execute(Job job) {
        if (job != null) {
            synchronized (jobs) {
                jobs.addLast(job);
                jobs.notify();
            }
        }
    }
    @Override
    public void shutdown() {
        for (Worker worker : workers) {
            worker.shutdown();
        }
    }
    class Worker implements Runnable {
        private volatile boolean running = true;
        @Override
        public void run() {
            while (running) {
                Job job = null;
                synchronized (jobs) {
                    while (jobs.isEmpty() && running) {
                        try {
                            jobs.wait();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                    if (!jobs.isEmpty()) {
                        job = jobs.removeFirst();
                    }
                }
                if (job != null) {
                    try {
                        job.execute();
                    } catch (Exception ignored) {
                    }
                }
            }
        }
        public void shutdown() {
            running = false;
            synchronized (jobs) {
                jobs.notifyAll();
            }
        }
    }
}
// ==================== Web 服务器 ====================
public class SimpleHttpServer {
    // 线程池
    private static ThreadPool threadPool;
    // 服务端套接字
    private static ServerSocket serverSocket;
    // 服务器根路径（静态资源目录）
    private static String basePath;
    // 服务器端口号
    private static int port;
    // 并发设置端口号与根路径，并判断线程池是否开启
    public static void start(int port, String path) {
        SimpleHttpServer.port = port;
        SimpleHttpServer.basePath = path;
        // 判断线程池是否已开启，未开启则初始化
        if (threadPool == null) {
            threadPool = new SimpleThreadPool(10);
            System.out.println("线程池已启动，核心线程数: 10");
        }
        try {
            serverSocket = new ServerSocket(port);
            System.out.println("服务器已启动，端口: " + port + "，根路径: " + basePath);
            // 主循环：不断接收客户端连接
            while (true) {
                Socket socket = serverSocket.accept();
                // 将请求封装为任务，提交给线程池异步处理
                threadPool.execute(new HttpRequestHandler(socket));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    // 请求处理任务
    static class HttpRequestHandler implements Job {
        private final Socket socket;
        public HttpRequestHandler(Socket socket) {
            this.socket = socket;
        }
        @Override
        public void execute() {
            try (
                BufferedReader reader = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
                PrintWriter writer = new PrintWriter(socket.getOutputStream())
            ) {
                // 解析请求行（GET /index.html HTTP/1.1）
                String requestLine = reader.readLine();
                if (requestLine == null || requestLine.isEmpty()) {
                    sendError(writer, 400, "Bad Request");
                    return;
                }
                String[] parts = requestLine.split(" ");
                if (parts.length < 3) {
                    sendError(writer, 400, "Bad Request");
                    return;
                }
                String resourcePath = parts[1];  // 请求的资源路径
                if (resourcePath.equals("/")) {
                    resourcePath = "/index.html";  // 默认首页
                }
                // 构建完整文件路径
                File file = new File(basePath, resourcePath);
                // 判断资源状态并返回对应响应
                if (!file.exists()) {
                    sendError(writer, 404, "Not Found");           // 资源不存在
                } else if (!file.canRead()) {
                    sendError(writer, 403, "Forbidden");           // 无权限访问
                } else if (file.isDirectory()) {
                    sendError(writer, 403, "Forbidden");           // 目录访问禁止
                } else {
                    sendFile(writer, file);                         // 200 OK，返回文件内容
                }
            } catch (IOException e) {
                // 服务器内部错误
                try (PrintWriter writer = new PrintWriter(socket.getOutputStream())) {
                    sendError(writer, 500, "Internal Server Error");
                } catch (IOException ignored) {
                }
            } finally {
                try {
                    socket.close();
                } catch (IOException ignored) {
                }
            }
        }
        // 返回文件内容（200 OK）
        private void sendFile(PrintWriter writer, File file) throws IOException {
            writer.println("HTTP/1.1 200 OK");
            writer.println("Content-Type: text/html; charset=UTF-8");
            writer.println("Content-Length: " + file.length());
            writer.println();
            writer.flush();
            // 输出文件内容
            try (BufferedReader fileReader = new BufferedReader(new FileReader(file))) {
                String line;
                while ((line = fileReader.readLine()) != null) {
                    writer.println(line);
                }
            }
            writer.flush();
        }
        // 返回错误响应
        private void sendError(PrintWriter writer, int statusCode, String message) {
            writer.println("HTTP/1.1 " + statusCode + " " + message);
            writer.println("Content-Type: text/html; charset=UTF-8");
            writer.println();
            writer.println("<h1>" + statusCode + " " + message + "</h1>");
            writer.flush();
        }
    }
    // 停止服务器
    public static void stop() {
        if (threadPool != null) {
            threadPool.shutdown();
        }
        try {
            if (serverSocket != null) {
                serverSocket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    // ==================== 启动入口 ====================
    public static void main(String[] args) {
        // 并发设置端口号与根路径，内部自动判断线程池是否开启
        SimpleHttpServer.start(8080, "/var/www/html");
    }
}

### 三、核心设计要点

|设计点|实现方式|对应你的时序|
|---|---|---|
|==**端口号与根路径**==|`start(port, path)` 并发设置|阶段②|
|==**线程池状态检查**==|`if (threadPool == null)` 判断并初始化|阶段③|
|==**请求封装与提交**==|`threadPool.execute(new HttpRequestHandler(socket))`|阶段④|
|==**异步处理**==|`HttpRequestHandler` 实现 `Job` 接口，由工作者线程执行|阶段⑤|
|==**状态码返回**==|根据资源是否存在、是否有权限，返回 200/403/404/500|阶段⑤|
|==**响应输出**==|`PrintWriter` 将响应内容发送给客户端|阶段⑥|

### 四、面试要点

**Q：为什么主线程接收连接后要立即交给线程池？**

主线程只负责接收 TCP 连接，如果同步处理请求（读取文件、生成响应），处理期间无法接收新连接，并发能力极低。交给线程池异步处理后，主线程可以立刻回去 `accept()` 下一个连接，多个工作者线程并行处理请求，吞吐量大幅提升。

**Q：这个实现和你之前写的连接池有什么共同点？**

两者都使用了池化思想——线程池复用线程，连接池复用数据库连接。核心机制完全相同：都是基于生产者-消费者模式，都有工作队列，都使用 `wait/notifyAll` 实现等待与唤醒。你把这两个示例对比着看，池化技术的通用设计模式就完全掌握了。