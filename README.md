hugo --minify
hugo --minify --destination=./docs

git clone --recurse-submodules https://github.com/udarabibile/code-koala-blog.git
git submodule update --remote themes/hugo-future-imperfect-slim

git config user.name "Udara Bibile"
git config user.email "udarabibile94@gmail.com"

git reset --soft HEAD~1
