#!groovy

node('master'){

	def remote_host = "34.228.196.96"
	def git_repo_name = "github.com/rafioul/ansible-code.git"

	stage ('Buid Repository') {
		withCredentials([sshUserPrivateKey(credentialsId: "git-ssh-key", keyFileVariable: 'keyfile')]) {
			echo "keyfile: ${keyfile}"
			sh "scp -i ~/.ssh/grafana.pem ${keyfile} centos@${remote_host}:/home/centos/.ssh/id_rsa"
			sh """ssh -i ~/.ssh/grafana.pem centos@${remote_host} << EOF
sudo rm -rf /opt/ghost
sudo mkdir -p /opt/ghost
sudo mkdir -p /opt/previous-release
sudo mkdir -p /opt/current-release

cd /opt/ghost

sudo ssh-agent bash -c 'ssh-add /home/centos/.ssh/id_rsa; git clone git@github.com:rafioul/ansible-code.git .'

sudo mkdir -p /opt/releases/ghost-\"\$(cd /opt/ghost && git log --format="%H" -n 1)\"
sudo cp -R /opt/ghost/* /opt/releases/ghost-\"\$(cd /opt/ghost && git log --format="%H" -n 1)\"
sudo chmod -R +x /opt/releases/ghost-\"\$(cd /opt/ghost && git log --format="%H" -n 1)\"
sudo rm -rf /opt/previous-release/*
sudo ln -sfn \"\$(readlink -f /opt/current-release)\" /opt/previous-release
sudo service nginx stop
sudo rm -rf /opt/current-release/*
sudo ln -sfn /opt/releases/ghost-\"\$(cd /opt/ghost && git log --format="%H" -n 1)\"/* /opt/current-release
sudo rm -rf /var/www/ghost/*
sudo ln -sfn /opt/current-release/* /var/www/ghost/
cd /var/www/ghost/
# yarn
sudo ln -sf /var/www/ghost/system/files/ghost.audiomack.com.conf /etc/nginx/conf.d/ghost.audiomack.com.conf
sudo service nginx restart

if ./start.sh; then
	echo "Build successful and doing clean-up"
	sudo ls -dt /opt/releases/ghost-*/ | tail -n +6 | xargs rm -rf
else
	echo "Build unsuccessful and starting rollback process"
	sudo service nginx stop
	sudo rm -rf /opt/current-release/*
	links=`readlink -f /opt/current-release/*`
	sudo ln -sfn \"\$(readlink -f /opt/current-release)\" /opt/current-release
	sudo service nginx restart
fi
EOF
"""
		}
	}
}
