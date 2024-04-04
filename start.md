启动服务分为四步：

1、启动中间件
cd docker
rm -rf volumes
sudo docker compose -f docker-compose.middleware.yaml up -d


2、启动flask
screen -S flask  -L  -Logfile flask.log
cd api
cp .env.example .env
openssl rand -base64 42
sed -i 's/SECRET_KEY=.*/SECRET_KEY=light/' .env
pip install -r requirements.txt
flask db upgrade
flask run --host 0.0.0.0 --port=5001 --debug

3、启动消息队列
screen -S celery  -L  -Logfile celery.log
cd api
celery -A app.celery worker -P gevent -c 1 -Q dataset,generation,mail --loglevel INFO


4、启动web
screen -S web  -L  -Logfile web.log

node --version   //确认node的版本是18.19.0
cd web
npm install
npm run dev

模型服务启动分为两步

1、xinference 启动：
screen -S xinf  -L  -Logfile xinf.log
conda activate xinf
XINFERENCE_MODEL_SRC=modelscope xinference-local --host 0.0.0.0 --port 8085
2、打开web页面后启动 embedding和reranking


其他操作：
nvm 设置默认版本
nvm alias default v18.19.0


systemctl daemon-reload
systemctl restart frpc