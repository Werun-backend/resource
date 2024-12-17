# Gitlet

在这个项目中，您将实现一个版本控制系统，该系统模仿流行系统 Git 的一些基本功能。然而，我们的系统更小、更简单，因此我们将其命名为 Gitlet。

版本控制系统本质上是相关文件集合的备份系统。Gitlet 支持的主要功能包括：

1. 保存整个文件目录的内容。在 Gitlet 中，这称为_提交_，保存的内容本身称为_提交_。
2. 恢复一个或多个文件或整个提交的版本。在 Gitlet 中，这称为_检出_这些文件或该提交。
3. 查看备份历史记录。在 Gitlet 中，您可以在日志中查看此历史_记录_。
4. 维护相关的提交序列，称为_分支_。
5. 将一个分支中所做的更改合并到另一个分支中。

详细文档 ：https://sp21.datastructur.es/materials/proj/proj2/proj2#overview-of-gitlet

框架代码：[https://github.com/Berkeley-CS61B/skeleton-sp21](https://github.com/Berkeley-CS61B/skeleton-sp21)

# 1. 序列化与逆序列化

**持久化程序状态**

实现这一目标的关键思想是使用计算机的文件系统。通过将信息存储到硬盘驱动器，程序能够留下信息以供以后执行使用。-----序列化

**序列化** ：序列化是将对象转换为一系列字节的过程，然后可以将其存储在文件中。然后，我们可以_反序列化_这些字节，并在将来调用程序时取回原始对象。

java 序列化

1.实现接口 `java.io.Serializable` 接口

`Serializable` 是一个标记接口

2.**序列化的核心类**：

通过序列化，可以实现数据持久化、远程传输等功能。

```java
eg 序列化
Model m = ....;
File outFile = new File(saveFileName);
try {
ObjectOutputStream out =
        new ObjectOutputStream(new FileOutputStream(outFile));
    out.writeObject(m);
    out.close();
    //将m序列化存储在 savefileName中
} catch (IOException excp) {
        ...
        }
```

gitlet 在 utils 类中封装好了序列化和反序列化方法

项目中主要为 Blob & commit & branch stage repo 需要被磁盘存储的类编写序列化和逆序列化方法

# SHA-1 哈希值

使用名为 SHA-1 的_加密哈希函数_，该函数可从任何字节序列生成 160 位整数哈希。加密哈希函数具有以下特性：很难找到两个具有相同哈希值的不同字节流（或者实际上很难找到_任何_仅给定其哈希值的字节流），因此，本质上，我们可以假设任何两个具有不同内容的对象具有不同相同 SHA-1 哈希值

在 gitlet 中有

```java
static String sha1(Object... vals) {
    try {
        MessageDigest md = MessageDigest._getInstance_("SHA-1");
        for (Object val : vals) {
            if (val instanceof byte[]) {
                md.update((byte[]) val);
            } else if (val instanceof String) {
                md.update(((String) val).getBytes(StandardCharsets._UTF_8_));
            } else {
                throw new IllegalArgumentException("improper type to sha1");
            }
        }
        Formatter result = new Formatter();
        for (byte b : md.digest()) {
            result.format("%02x", b);
        }
        return result.toString();
    } catch (NoSuchAlgorithmException excp) {
        throw new IllegalArgumentException("System does not support SHA-1");
    }
}
```

可以生成 sha-1 哈希值

两个具有不同内容的对象具有不同相同 SHA-1 哈希值这使我们**仅需要比较哈希值就可以分别不同提交的文件是否发生了改变**

1 唯一标识提交、文件、目录、对象等；

2 确保数据的完整性；

3 **高效地存储和查找对象**；

4 **支持分支和合并操作。**

通过为对象编写通过 sha-1 值序列化，逆序列化方法

```java
public String write() {
    if (!_COMMIT_DIR_.exists()) {
        _COMMIT_DIR_.mkdir();
    }
    String sha1 = Utils._sha1_(Utils._serialize_(this));
    File commitFile = _join_(_COMMIT_DIR_, sha1);
    if (!commitFile.exists()) {
        try {
            commitFile.createNewFile();
        } catch (IOException e) {
            System._err_.println("写入提交文件时出错: " + e.getMessage());
        }
    }
    Utils._writeObject_(commitFile, this);
    return sha1;
}

_/** 根据 SHA-1 读取提交_
_ *_
_ *_
_ * */_
public static Commit read(String sha1) {
    File commitFile = _join_(_COMMIT_DIR_, sha1);
    if (!commitFile.exists()) {
        return null;
    }
    return Utils._readObject_(commitFile, Commit.class);
}
```

实现了类似内存指针的对外存中的对象的存储与访问

# Blob & commit & branch

SHA-1 哈希值与文件存在一一对应关系

![](static/DaOpbjsH0oUAUpxHgTIcaybGnLd.png)

blob 对象包含了文件的内容和该内容对应的哈希值

commit 对象中保存了 blob 中的哈希值，实现不同 commit 指向 blob 对应不同版本的内容

Branch 保存该分支最新的 commit 的哈希值

Blob，commit，branch 都被序列化存储

![](static/OsERbcxHxoBGOjxVXUMcDMk6nsd.png)

# Gitlet 命令

## 2.1 init

## 2.2 add

## 2.3 commit

1. **读取旧提交和暂存区**

- **当前提交 (****head****)：** 通过 `head` 引用获取当前提交对象。
- 复制代码
- `Commit oldCmt = Commit.read(head);`
- **暂存区 (****stage****)：** 读取暂存区的状态。
- 复制代码
- `Stage stage = new Stage();`

1. 将旧提交与暂存区合并

- Map<String,String> newMap = Commit._mergeBlobs_(stage,oldCommit);

3.创建新提交并持久化存储

## 2.4 merge

![](static/FdZcbBxwdoikPnxuZufcVyymnUg.png)

### **合并操作的基本步骤**

1. **检查当前分支和目标分支的差异**

   - Gitlet 会比较当前分支（HEAD 所指向的分支）和要合并的目标分支（`branchName`）之间的差异。
2. **查找最近的共同祖先**

   - 计算当前分支和目标分支的**共同祖先**（Common Ancestor）。共同祖先是指两个分支分开之前的最后一个提交。
   - 如果分支合并后没有冲突，Gitlet 会将文件内容从共同祖先到当前分支和目标分支的改动合并。
3. **三方合并**

   - Gitlet 通过“三方合并”（three-way merge）来合并文件，即通过当前分支、目标分支和共同祖先三个版本的内容来进行比较和合并。
   - **当前分支**：HEAD 所指向的提交。
   - **目标分支**：要合并的分支。
   - **共同祖先**：两个分支的分支点。
4. **处理冲突**

   - 如果两个分支在同一个文件的相同部分做了不同修改，那么就会发生**冲突**。Gitlet 会在文件中标记冲突部分，通常用如下格式：
   - `<<<<<<< HEAD 当前分支的内容 ======= 目标分支的内容 ``>>>>>>> <branchName>`
   - 用户需要手动解决冲突，决定保留哪部分内容，或者合并修改后的内容。
5. **创建新的合并提交（Merge Commit）**

   - 在解决完所有冲突后，Gitlet 会创建一个新的合并提交（`Merge Commit`）。该提交的父提交是当前分支的提交和目标分支的提交。
6. **更新 HEAD 和分支指针**

   - 合并成功后，HEAD 会指向新的合并提交，当前分支会被更新到这个合并后的提交。

# Remote 的实现

。。。。
