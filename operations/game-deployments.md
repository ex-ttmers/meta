## Game teaching Deployments

[Mathbot way](#mathbot-way--deploys-all-games-)

[Jenkins way](#jenkins-way--allows-single-or-all-game-engine-deployment-)

[Manual way](#manual-way)

### Mathbot way ( Deploys ALL games )

`mathbot deploy games staging rc`

or

`mathbot deploy games production rc`

__NOTE:__ We are deploying rc to production here purposely.  The games are dependent on bowser, and bower requires a branch specified, and we check in the bower.json file, so we just deploy rc to staging and to production.  This is an opportunity for improvement, but because games are so rarely updated, it has not been worth the time.

### Jenkins way ( allows single or all game engine deployment )

1. Goto http://jenkins.thinkthroughmath.com:8080/job/Deploy-games/
2. click "Build with Parameters"
3. select the appropriate environment.
4. select RC as the branch. We only deploy the rc branch to staging **AND** production because of bower dependencies noted above.
6. Add your mention name to be pinged in chat about deployment progress.
7. Click "Build"
8. Wait to be pinged in chat

### Manual way

1. Clone [GG-server](https://github.com/thinkthroughmath/GG-Server)

        git clone git@github.com:thinkthroughmath/GG-Server.git
        
2. Setup GG-Server

        cd gg-server
        npm install -g bower; npm install -g grunt-cli; npm install
        
3. Configure for deployment
        
        FARM="production"
        AWS_SECRET=$(ttmscalr config: TTM_AWS_ACCESS_KEY -f ${FARM})
        AWS_KEY=$(ttmscalr config: TTM_AWS_ACCESS_KEY_ID -f ${FARM})
        AWS_BUCKET=$(ttmscalr config: TTM_AWS_BUCKET -f ${FARM})

        SERVER=https://lms.thinkthroughmath.com
        API_ACCESS_KEY=$(ttmscalr config:get TTM_API_SECRET_TOKEN -f ${FARM})

        echo "{ \"key\": \"${API_ACCESS_KEY}\", \"server\": \"${SERVER}\" }" > .apangea.json
        echo "{ \"key\": \"${AWS_KEY}\", \"secret\": \"${AWS_SECRET}\", \"bucket\": \"${AWS_BUCKET}\" }" > .aws.json
        
4. Ready deployment

        npm install
        grunt ready-deploy
        
5. Deploy

        ENGINE="NAME OF THE GAME ENGINE YOU WOULD LIKE TO DEPLOY, IN LOWER CASE"
        grunt shell:deploy-$ENGINE
        
6. Update Apangea Game Version references

This updates GameAssetsVersion in the Apangea database

        grunt test-all-deploy-api-calls
