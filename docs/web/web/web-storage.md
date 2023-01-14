> Indexed DB is considered more powerful than `localStorage!`

![image-20230107135823659](https://img-note.langyastudio.com/202301071358004.png?x-oss-process=style/watermark)



- Even though it has modern browser support, browsers such as IE does not have complete support for this

![](https://img-note.langyastudio.com/202301071358513.png?x-oss-process=style/watermark)



- **Can store much bigger volumes of data than** **` localStorage`**

There is no particular limit like in `localStorage` (between 2.5MB and 10MB). The maximum limit is based on the browser and the disk space. For example, Chrome and Chromium-based browsers allow up to 80% disk space. If you have 100GB, Indexed DB can use up to 80GB of space, and 60GB by a single origin. Firefox allows up to 2GB per origin while Safari allows up to 1GB per origin



## 参考

[How to Use IndexedDB — A NoSQL DB on the Browser](https://blog.bitsrc.io/how-to-use-indexeddb-a-nosql-db-on-the-browser-f845da3caf35)

[Maximum item size in IndexedDB](https://stackoverflow.com/questions/5692820/maximum-item-size-in-indexeddb)



