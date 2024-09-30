# connect-log-athena-query


## DDL in Athena

- raw data (bc4.json) mapping table b_connect 

```SQL
CREATE EXTERNAL TABLE `b_connect`(
  `Version` string, 
  `AWSAccountId` string, 
  `InstanceId` string, 
  `InitialContactId` string, 
  `ContactId` string, 
  `DisplayName` string, 
  `Participants` array<struct<ParticipantId:string>>,
  `Transcript` array<struct<AbsoluteTime:string,Content:string,ContentType:string,Id:string,Type1:string,ParticipantId:string,DisplayName:string,ParticipantRole:string>>
  )
ROW FORMAT SERDE 
  'org.openx.data.jsonserde.JsonSerDe' 
LOCATION
  's3://<bucket name>/b/'
```

- relationalized table for field 'Transcript' named as b_relationalize_full_transcript
- same operation could be done towards field 'Participants'

```SQL
create external table `b_relationalize_full_transcript` (
  `Version` string, 
  `AWSAccountId` string, 
  `InstanceId` string, 
  `InitialContactId` string, 
  `ContactId` string, 
  `AbsoluteTime` string, 
  `Content` string, 
  `ContentType` string, 
  `Id` string, 
  `ParticipantId` string, 
  `DisplayName` string, 
  `ParticipantRole` string
)
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.OpenCSVSerde' 
LOCATION
  's3://<bucket name>/b_relationalize_full_transcript/'
```


## DML in Athena

- insert data from raw data table into relationalized table

```SQL
insert into default.b_relationalize_full_transcript
select Version,
       AWSAccountId,
       InstanceId,
       InitialContactId,
       ContactId,
       transcript[1].AbsoluteTime,
       transcript[1].Content,
       transcript[1].ContentType,
       transcript[1].Id,
       transcript[1].ParticipantId,
       transcript[1].DisplayName,
       transcript[1].ParticipantRole
from default.b_connect


insert into default.b_relationalize_full_transcript
select Version,
       AWSAccountId,
       InstanceId,
       InitialContactId,
       ContactId,
       transcript[2].AbsoluteTime,
       transcript[2].Content,
       transcript[2].ContentType,
       transcript[2].Id,
       transcript[2].ParticipantId,
       transcript[2].DisplayName,
       transcript[2].ParticipantRole
from default.b_connect

insert into default.b_relationalize_full_transcript
select Version,
       AWSAccountId,
       InstanceId,
       InitialContactId,
       ContactId,
       transcript[3].AbsoluteTime,
       transcript[3].Content,
       transcript[3].ContentType,
       transcript[3].Id,
       transcript[3].ParticipantId,
       transcript[3].DisplayName,
       transcript[3].ParticipantRole
from default.b_connect

... ...
```

- data query show case
<img width="2242" alt="Screenshot 2024-09-30 at 10 53 59" src="https://github.com/user-attachments/assets/913d76c5-51f2-49fe-bf81-c87861862568">

