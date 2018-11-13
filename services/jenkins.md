# Jenkins

<toc>
- [Jenkins](#jenkins)
- [Minions](../documentation/jenkins-minions.md)
- [Troubleshooting](#troubleshooting)
</toc>

# Troubleshooting

## java.lang.IllegalArgumentException: byteString == null

If you see this, then the github cache the jenkins keeps has been corrupted. To fix, you just need to delete the cache.

    ttmscalr run jenkins.1 -f jenkins -c "rm -rf /jenkins/org.jenkinsci.plugins.github.GitHubPlugin.cache/"
