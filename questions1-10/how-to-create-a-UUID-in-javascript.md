原问题：[How to create a GUID / UUID in Javascript?](http://stackoverflow.com/questions/105034/how-to-create-a-guid-uuid-in-javascript)

## Introuction
UUID = Universally Unique IDentifier, 全球唯一标识符。

本来我觉得UUID不是什么事，只是一个唯一性的ID字符串而已。在Stack Overflow上看到这个问题之后，倒开始觉得，也许UUID里也有一些值得去发掘的东西。

于是我想到了这几个问题：

* [为什么有UUID和GUID这两个东西](#uuid-and-guid)
* [有什么规范吗](#specification)
* 好像我曾多次看到过有人讨论某种系统或变成语言下UUID的生成问题，[UUID跟系统或者编程语言有关系吗](#uuid-and-implementation)

带着这些问题，我查找并学习了一番。现在做个整理。

## UUID and GUID
#### 定义
UUID来自于IETF发布的一个规范：[A Universally Unique IDentifier (UUID) URN Namespace](http://www.ietf.org/rfc/rfc4122.txt)。
> This specification defines a Uniform Resource Name namespace for UUIDs (Universally Unique IDentifier), also known as GUIDs (Globally Unique IDentifier).  A UUID is 128 bits long, and can guarantee uniqueness across space and time.  UUIDs were originally used in the Apollo Network Computing System and later in the Open Software Foundation's (OSF) Distributed Computing Environment (DCE), and then in Microsoft Windows platforms.
This specification is derived from the DCE specification with the kind permission of the OSF (now known as The Open Group).

UUID和GUID是同一个东西的两个名字。这两个名字的来源不同。

* UUID来源于OSF的DCE规范，也就是RFC4122的前身
* GUID来源于微软，注意RFC4122的作者之一是微软员工

#### 用途
UUID的出现，是为了在一个复杂的系统中，唯一的标识每个信息实体，同时不需要有一个集中的id管理。也就是说，根据某种规则来为一个信息实体分配一个唯一的id，而且不需要一个id管理器来保证这个id的唯一性。

它可以用来标识任何东西，Microsoft用它来表示Windows中的软件（GUID），Linux用它来表示系统中的文件。

## Specification
#### UUID格式规范
这128bits的结构如下所示：
```
   0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                          time_low                             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |       time_mid                |         time_hi_and_version   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |clk_seq_hi_res |  clk_seq_low  |         node (0-1)            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         node (2-5)                            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      time-low              = 4hexOctet
      time-mid              = 2hexOctet
      time-high-and-version  = 2hexOctet    // MOST IMPORTANT !
      clock-seq-and-reserved = hexOctet
      clock-seq-low          = hexOctet
      node                   = 6hexOctet
      hexOctet               = hexDigit hexDigit
      hexDigit =
            "0" / "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" /
            "a" / "b" / "c" / "d" / "e" / "f" /
            "A" / "B" / "C" / "D" / "E" / "F"
```
示例
> uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6

#### UUID生成算法
UUID本身也经过了[多个版本的演化](http://en.wikipedia.org/wiki/Universally_unique_identifier#Variants_and_versions)。比如node字段的定义，最开始定义为来自IEEE 802 address，演化到后来的*a randomly or pseudo-randomly generated 48-bit value*。

最新的UUID生成算法是这样的：

 * Set the two most significant bits (bits 6 and 7) of the clock_seq_hi_and_reserved to zero and one, respectively.
 * Set the four most significant bits (bits 12 through 15) of the time_hi_and_version field to the 4-bit version number.
 * Set all the other bits to randomly (or pseudo-randomly) chosen values.

## UUID and implementation
越来越多的系统中使用着UUID，各自使用目的并不一样。而且由于限定在128bits，UUID规范本身并没有保证UUID真的是在全球唯一的。因此现在对UUID的使用，一般都是限定在一个范围内有唯一性保证，比如一个操作系统内。

因此我们可以看到：

* 微软有一个GUID生成lib：http://msdn.microsoft.com/en-us/library/system.guid(v=vs.110).aspx
* Linux也同样有UUID生成lib：http://en.wikipedia.org/wiki/Util-linux
* Android的UUID生成lib：http://developer.android.com/reference/java/util/UUID.html

用以上系统对应的UUID生成lib可以确保产生的UUID在系统范围内是唯一的。因此可以用来标识系统资源，比如文件、软件、设备等。

而对于某种具体的编程语言，UUID的使用并没有必要。因为使用UUID的目的是给某个资源分配一个在当前环境下唯一的标识符。而一个程序只有在运行的时候才谈得上有一个**环境**，进程之间又互不影响。因此，一般在编程语言规范中并没有对UUID生成方法进行规定。

但*unique id*还是很常用的，比如：

* PHP - uniqid() http://php.net/manual/en/function.uniqid.php#94959
* Mysql - UUID() http://dev.mysql.com/doc/refman/5.1/en/miscellaneous-functions.html#function_uuid
* Java - UUID http://docs.oracle.com/javase/7/docs/api/java/util/UUID.html

但这些UUID方法只是借用了*唯一性*的概念，并不是必须。而且uuid规定128bits，很多情况下有点浪费。

你可以用任何方法来生成一个程序内唯一的字符串，比如mysql中的incremental id，它就比uuid实用。

## UUID in Javascript

既然要生成UUID，那么得符合规范。

* Javascript function
```
function generateUUID(){
    var d = new Date().getTime();
    var uuid = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        var r = (d + Math.random()*16)%16 | 0;
        d = Math.floor(d/16);
        return (c=='x' ? r : (r&0x7|0x8)).toString(16);
    });
    return uuid;
};
```
* Node module
https://github.com/broofa/node-uuid