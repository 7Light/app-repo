# nginx-with-fancy
The docker for repo website and app-publish service
app-publish : docker run -it -p {server_port}:80 -v {server_repo_path}:/repo {docker}