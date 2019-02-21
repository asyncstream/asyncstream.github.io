---
layout: post
title:  "Serverless With Nodejs & Sequelize"
---

# Serverless With Nodejs & Sequelize

If you are searching for a solution to setup a Nodejs AWS Lamda function communicates to MySQL then follow the below steps.

## Setup

1. Install Nodejs
2. Setup Serverless (offline mode)

        npm install -g serverless
    
    I chose the offline emulation to speed up the development. 
        
        npm install serverless-offline --save-dev

3. Configure package.json

        {
            "name": "community-crud-service",
            "version": "0.0.1",
            "description": "Community Crud Service",
            "main": "index.js",
            "dependencies": {
                "mysql2": "^1.6.4",
                "node-uuid": ">= 1.4.1",
                "sequelize": "^4.42.0"
            }
        }

4. Configure serverless.yml

        service: community-crud-service

        custom:
        secrets: ${file(secrets.json)}

        provider:
        name: aws
        runtime: nodejs8.10
        timeout: 30
        stage: ${self:custom.secrets.NODE_ENV}
        region: sa-east-1
        environment: 
            NODE_ENV: ${self:custom.secrets.NODE_ENV}
            DB_NAME: ${self:custom.secrets.DB_NAME}
            DB_USER: ${self:custom.secrets.DB_USER}
            DB_PASSWORD: ${self:custom.secrets.DB_PASSWORD}
            DB_HOST: ${self:custom.secrets.DB_HOST}
            DB_PORT: ${self:custom.secrets.DB_PORT}
        vpc:
            securityGroupIds:
            - ${self:custom.secrets.SECURITY_GROUP_ID}
            subnetIds:
            - ${self:custom.secrets.SUBNET1_ID}
            - ${self:custom.secrets.SUBNET2_ID}
            - ${self:custom.secrets.SUBNET3_ID}
            - ${self:custom.secrets.SUBNET4_ID}

        functions:
        healthCheck:
            handler: index.healthCheck
            events:
            - http:
                path: /
                method: get
                cors: true
        create:
            handler: index.create
            events:
            - http:
                path: community
                method: post
                cors: true
        getOne:
            handler: index.getOne
            events:
            - http:
                path: community/{handlecode}
                method: get
                cors: true
        update:
            handler: index.update
            events:
            - http:
                path: community/{handlecode}
                method: put
                cors: true
        destroy:
            handler: index.destroy
            events:
            - http:
                path: community/{handlecode}
                method: delete
                cors: true

        plugins:
        - serverless-offline


    The secrets.json file contains the db connection values

        {
            "DB_NAME": "community",
            "DB_HOST": "localhost"
        }

5. Create a js file which defines the entities in sequelize

        'use strict'
        module.exports.community = (sequelize, type) => {
            return sequelize.define('t_community', {
            handlecode: {
                type: type.STRING,
                primaryKey: true
            },
            name: type.STRING,
            description:{
                type: type.STRING,
                allowNull: true,
            },
            status: type.STRING,
            registered_on: type.DATE,
            ceased_on: {
                type: type.DATE,
                allowNull: true,
            },
            creation_date: { type: type.DATE, defaultValue: type.NOW },
            created_by: type.STRING,
            last_updation_date: { type: type.DATE, defaultValue: type.NOW },
            last_updated_by: type.STRING,
            },{
                timestamps: false,
                underscored: true,
                freezeTableName: true,
            })
        }

        module.exports.community_address = (sequelize, type) => {
            return sequelize.define('t_community_address', {
                id: {
                type: type.INTEGER,
                primaryKey: true,
                autoIncrement: true
                },
                community: type.STRING,
                version: {type:type.INTEGER,defaultValue:0},
                geo_address:type.INTEGER,
                address_line_1: type.STRING,
                house_number: type.STRING,
                last_updation_date: { type: type.DATE, defaultValue: type.NOW },
                last_updated_by: type.STRING
            },{
                timestamps: false,
                underscored: true,
                freezeTableName: true,
            })
        }

        module.exports.community_contact = (sequelize, type) => {
            return sequelize.define('t_community_contact', {
                id: {
                type: type.INTEGER,
                primaryKey: true,
                autoIncrement: true
                },
                community: type.STRING,
                contact_type:type.STRING,
                value_type: type.STRING,
                value: type.STRING,
                deletion_status: {type:type.INTEGER,defaultValue:0},
                last_updation_date: { type: type.DATE, defaultValue: type.NOW },
                last_updated_by: type.STRING
            },{
                timestamps: false,
                underscored: true,
                freezeTableName: true,
            })
        }


        module.exports.geo_address = (sequelize, type) => {
            return sequelize.define('t_geo_address', {
                id: {
                type: type.INTEGER,
                primaryKey: true,
                autoIncrement: true
                },
                country: type.STRING,
                zipcode: type.STRING,
                state: type.STRING,
                city: type.STRING
            },{
                timestamps: false,
                underscored: true,
                freezeTableName: true,
            })
        }

6. Database connection setup (db.js)

        Create a script which handles database communication

        const Sequelize = require('sequelize')
        const communityEntities = require('./models/CommunityEntities')

        const sequelize = new Sequelize(
        process.env.DB_NAME,
        process.env.DB_USER,
        process.env.DB_PASSWORD,
        {
            dialect: 'mysql',
            host: process.env.DB_HOST,
            port: process.env.DB_PORT,
            // disable logging; default: console.log
            logging: false
        }
        )

        const CommunityEntity = communityEntities.community(sequelize, Sequelize)
        const CommunityAddressEntity = communityEntities.community_address(sequelize, Sequelize)
        const CommunityContactEntity = communityEntities.community_contact(sequelize, Sequelize)
        const GeoAddressEntity = communityEntities.geo_address(sequelize, Sequelize)
        const Models = { CommunityEntity,CommunityAddressEntity,CommunityContactEntity,GeoAddressEntity }
        const connection = {}
        CommunityEntity.hasOne(CommunityAddressEntity,{as: 'address', foreignKey : 'community'})
        CommunityEntity.hasMany(CommunityContactEntity,{as: 'contacts', foreignKey : 'community'})
        CommunityAddressEntity.belongsTo(CommunityEntity,{foreignKey: 'community', targetKey: 'handlecode'})
        CommunityContactEntity.belongsTo(CommunityEntity,{foreignKey: 'community', targetKey: 'handlecode'})
        CommunityAddressEntity.belongsTo(GeoAddressEntity,{as: 'geoaddress', foreignKey: 'geo_address', targetKey: 'id'})


        module.exports.sequelize = async () => {
        if (connection.isConnected) {
            console.log('=> Using existing connection.')
            return sequelize
        }

        //await sequelize.sync()
        await sequelize.authenticate()
        connection.isConnected = true
        console.log('=> Created a new connection.')  
        return sequelize
        }
        module.exports.models = async () => {
        if (!connection.isConnected) {
            await sequelize.authenticate()
            connection.isConnected = true
            console.log('=> Created a new connection.')  
        }
        return Models
        }

        // Find or Create the geo address
        module.exports.getOrCreateGeoAddress = async (geoaddress) => {
        return await GeoAddressEntity.findOrCreate({where: {zipcode: geoaddress.zipcode}, defaults: geoaddress}).spread(
            function(result, created){
            return result.dataValues
            }
        )
        }

    Now we have to create a community record with address and contacts in a atomic transaction

        module.exports.createCommunity = async (community,communityaddress,communitycontacts) => {
        return await sequelize.transaction(function (t) {
            return CommunityEntity.create(community, {transaction: t}).then(function () {
            return CommunityAddressEntity.create(communityaddress, {transaction: t}).then(function () {
                //return sequelize.Promise.each(communitycontacts, function(itemToCreate){
                return CommunityContactEntity.bulkCreate(communitycontacts, { transaction: t })
                //});
            }); 
            });  
        }).then(function (result) {
            return true;
        }).catch(function (err) {
            console.log(err)
            throw new Error();
        });
        }

        module.exports.getCommunity = async (communityId) =>{
        console.log(communityId)
        return await CommunityEntity.findOne({
            where: { handlecode: communityId },    
            include: [
            { model: CommunityAddressEntity, as: 'address', include: [
                {model: GeoAddressEntity, as: 'geoaddress'}
            ]},
            { model: CommunityContactEntity, as: 'contacts'}
            ]
        })
        }

7. Create lamda handler file (index.js).

        'use strict';
        const db = require('./db')
        const sequalize = db.sequelize
        const connectToDatabase = db.models
        const dtos = require('./dtos')

        console.log('Loading function');
        const Error500Msg = (err) =>({
            statusCode: err.statusCode || 500,
            headers: { 'Content-Type': 'text/plain' },
            body: err.message
        });
        const HTTPError = (statusCode, message)=>{
            const error = new Error(message)
            error.statusCode = statusCode
            return error
        };

        module.exports.healthCheck = async () => {
            await connectToDatabase()
            console.log('Connection successful.')
            return {
            statusCode: 200,
            body: JSON.stringify({ message: 'Connection successful.' })
            }
        }

        module.exports.create = async (event) => {
            try {
                const { CommunityEntity } = await connectToDatabase()
                const data=JSON.parse(event.body)        
                const auditdto=dtos.auditdto(Date.now(),"system",Date.now(),"system")
                const geoaddressdto = dtos.geoaddressdto(
                    data.address.baseAddress.country,data.address.baseAddress.pincode,
                    data.address.baseAddress.state,data.address.baseAddress.city
                )
                const geoaddress = await db.getOrCreateGeoAddress(geoaddressdto)
                
                const communitydto = dtos.communitydto(data.handleCode,data.name,data.description,'A',data.registeredOn)
                const communityWithAuditDto={...communitydto, ...auditdto}
                const communityaddressdto=dtos.communityaddressdto(
                    data.handleCode,geoaddress.id,data.address.streetLocation,
                    data.address.houseNumber,"system"
                )    
                const contacts = data.contacts
                var contactDtoArr=[]
                contacts.forEach(function(elem){
                    contactDtoArr.push(dtos.communitycontactdto(
                        data.handleCode,elem.contactType,elem.valueType,elem.value,"system"
                    ));
                })
                //await CommunityEntity.create(communityWithAuditDto)
                await db.createCommunity(communityWithAuditDto,communityaddressdto,contactDtoArr)        
                return {
                statusCode: 200,
                body: JSON.stringify({id:data.handleCode})
                }
            } catch (err) {   
                console.log(err)     
                return Error500Msg(new Error("Could not create the community"));
            }
        }


        module.exports.getOne = async (event) => {
            try {
                const { CommunityEntity } = await connectToDatabase()        
                const communityEntity = await db.getCommunity(event.pathParameters.handlecode)
                if (!communityEntity) throw new HTTPError(404, `Community with id: ${event.pathParameters.handlecode} was not found`)
                return {
                statusCode: 200,
                body: JSON.stringify(communityEntity)
                }
            } catch (err) {
                console.log(err)
                return Error500Msg(new Error('Could not fetch the Community.'));        
            }
        }

8. Run the command to start serverless in offline mode

        serverless offline --skipCacheInvalidation start

# Conclusion

With this short article I tried my best to create a small capsule of AWS Lamda service which might be helpful in your serveless project. 

Note: *While building this serverless service, I have referred the link https://dzone.com/articles/a-crash-course-on-serverless-with-aws-building-api.* 

**Hello world**, this is my first Jekyll blog post.

I hope you like it!
