---
layout: article
title: Gourse 使用
tags: []
key: 1d76ac9c-e7ab-437c-afa1-0b71b8f2be52
---

使用 Gourse 生成版本提交的历史记录视频.

<!--more-->

## 安装

Mac 上按照:

```bash
brew install gource
brew install ffmpeg
```

## 脚本

```bash
gource \
--path  path/to/repo \
--seconds-per-day 0.15 \
--title "parse5" \
-1280x720 \
--file-idle-time 0 \
--auto-skip-seconds 0.75 \
--multi-sampling \
--stop-at-end \
--highlight-users \
--hide filenames,mouse,progress \
--max-files 0 \
--background-colour 000000 \
--disable-bloom \
--font-size 24 \
--output-ppm-stream - \
--output-framerate 30 \
-o - \
| ffmpeg \
-y \
-r 60 \
-f image2pipe \
-vcodec ppm \
-i - \
-vcodec libx264 \
-preset ultrafast \
-pix_fmt yuv420p \
-crf 1 \
-threads 0 \
-bf 0 \
/path/to/video
```

## 多个仓库生成

```bash
#!/usr/bin/env bash
# Generates gource video (h.264) out of multiple repositories.
# Pass the repositories in command line arguments.
# Example:
# <this.sh> /path/to/repo1 /path/to/repo2

RESOLUTION="1600x1080"
outfile="gource.mp4"

i=0
for repo in $*; do
	# 1. Generate a Gource custom log files for each repo. This can be facilitated by the --output-custom-log FILE option of Gource as of 0.29:
	logfile="$(mktemp /tmp/gource.XXXXXX)"
	gource --output-custom-log "${logfile}" ${repo}
	# 2. If you want each repo to appear on a separate branch instead of merged onto each other (which might also look interesting), you can use a 'sed' regular expression to add an extra parent directory to the path of the files in each project:
	sed -i -E "s#(.+)\|#\1|/${repo}#" ${logfile}
	logs[$i]=$logfile
	let i=$i+1
done

combined_log="$(mktemp /tmp/gource.XXXXXX)"
cat ${logs[@]} | sort -n > $combined_log
rm ${logs[@]}

echo "Committers:"
cat $combined_log | awk -F\| {'print  $2'} | sort | uniq
echo "======================"

time gource $combined_log \
	-s 0.4 \
	-i 0 \
	-$RESOLUTION \
	--highlight-users \
	--highlight-dirs \
	--file-extensions \
	--hide mouse,filenames \
	--key \
	--stop-at-end \
	--output-ppm-stream - | ffmpeg -y  -r 60 -f image2pipe -vcodec ppm -i - -threads 0 -r 24000/1001 -b 6144k -bt 8192k -vcodec libx264 -pass 1 -flags +loop -me_method dia -g 250 -qcomp 0.6 -qmin 10 -qmax 51 -qdiff 4 -bf 16 -b_strategy 1 -i_qfactor 0.71 -cmp +chroma -subq 1 -me_range 16 -coder 1 -sc_threshold 40 -flags2 -bpyramid-wpred-mixed_refs-dct8x8+fastpskip -keyint_min 25 -refs 1 -trellis 0 -directpred 1 -partitions -parti8x8-parti4x4-partp8x8-partp4x4-partb8x8 -an $outfile
rm $combined_log
```
