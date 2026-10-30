# shuying-music

《疏影》唱片柜（`/music.html`）和首页右下角小唱片放的蔡依林音频：198 个文件，1.81 GB。网站代码在私有仓库 [jikang119-ctrl/shuying](https://github.com/jikang119-ctrl/shuying)。

**一直保持私有**：歌有版权，只给疏影自己用。

## 给部署的 AI

完整流程照 shuying 仓库 `DEPLOY.md`「从另一台电脑做完整套」：先更新站点，再传歌，再验收。这个仓库只管「传歌」，两种方式：

**A. 服务器直接从 GitHub 拉**（不占家里的上传流量）

```bash
df -h /        # 剩余要大于 6 GB（clone 连 git 数据约 3.6 GB，再拷一份 1.8 GB）
git clone --depth 1 https://github.com/jikang119-ctrl/shuying-music.git /root/shuying-music
#   Username 填 jikang119-ctrl，Password 填 GitHub token（fine-grained，Contents: Read-only，勾上 shuying 和 shuying-music）
#   看 Receiving objects 那行的速度：一直低于 1 MiB/s 就 Ctrl+C、rm -rf /root/shuying-music，改用 B
mkdir -p /var/www/shuying/music
cp /root/shuying-music/*.mp3 /root/shuying-music/*.m4a /var/www/shuying/music/
chown -R www-data:www-data /var/www/shuying/music && find /var/www/shuying/music -type f -exec chmod 644 {} +
ls /var/www/shuying/music | wc -l       # 要 198
rm -rf /root/shuying-music              # 拷完可以删
```

**B. 另一台电脑 clone 下来再 scp 上去**（家里网络，上传 1.8 GB）

```powershell
# --separate-git-dir 让 git 数据放到 D:\shuying-music.git，不留在 D:\shuying-music 里，不然 scp 要多传 1.8 GB
git clone --separate-git-dir=D:\shuying-music.git https://github.com/jikang119-ctrl/shuying-music.git D:\shuying-music
cd D:\projects\shuying                                         # shuying 仓库根目录
python -X utf8 tools/check_music.py --dir D:/shuying-music     # 要打印「齐了」
scp -r D:\shuying-music root@8.163.12.169:/var/www/shuying/music-upload
```

然后在服务器上：

```bash
cd /var/www/shuying
rm -f music-upload/.git music-upload/.gitattributes music-upload/README.md    # mv * 不带点开头的文件，先删掉
mkdir -p music && mv music-upload/* music/ && rmdir music-upload
chown -R www-data:www-data music && find music -type f -exec chmod 644 {} +
ls music | wc -l       # 要 198
```

## 这些文件是什么

- 文件名是 `sha1(歌名#版本号)` 前 10 位加原扩展名。哪个文件是哪首歌、哪张专辑，看 shuying 仓库的 `content/music-catalog.json`；网站读的是 `site/assets/data/music.json`
- 由 shuying 仓库的 `tools/music.py` 从两个压缩包整理出来：熊猫无损「蔡依林合集」195 首 rar + 《Pleasure》zip。只去了下载站塞的水印标签，没有重新编码：185 个 MP3（320k）、13 个 m4a（256k AAC）
- 服务器上放在 `/var/www/shuying/music/`，nginx 要进门才给（没进门 403）；拖进度条靠 nginx 自带的 Range

## 补了新歌

在 shuying 仓库里照 `DEPLOY.md`「唱片柜」一节末尾重新生成（`music.py --out D:/projects/shuying-music --prune` 等四条），然后：

```powershell
cd D:\projects\shuying-music
git add -A && git commit -m "补了 N 首" && git push
```

已有的文件名不会变，服务器上只要把新增的文件传上去。
