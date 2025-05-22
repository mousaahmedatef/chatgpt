USE [NCP_DEVTEAMDB]
GO
/****** Object:  StoredProcedure [dbo].[Emergency_Incident_GetAllInformationAndGuidance]    Script Date: 5/18/2025 1:34:56 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
 ALTER              PROCEDURE [dbo].[Emergency_Incident_GetAllInformationAndGuidance]   
@IncidentKey NVARCHAR(250),
@DoneReplyStatus INT,
@WaitingReplyStatus INT,
@WaitingApprovalStatus INT,
@WaitingEditingStatus INT,
@WaitingPostStatus INT,
@InformationPostedStatus INT,
@CloseStatus INT,
@WaitingEntitiesStatus INT,
@GuidanceDone INT,
@EntityId INT = NULL,
@IsLeadership BIT = 0
AS
BEGIN
SELECT
	*
FROM (SELECT
		gui.[Id]
	   ,gui.[GuidanceKey] AS RequestKey
	   ,gui.[GuidanceDate] AS RequestDate
	   ,gui.[Description]
	   ,gui.[FkGuidanceTypeId] AS TypeId
	   ,gt.Name AS TypeName
	   ,gui.[FkIncidentId]
	   ,gui.[CreationDate]
	   ,gui.[CreationUserId]
	   ,gui.[LastUpdateDate]
	   ,gui.[LastUpdateUserId]
	   ,gui.[FkStatusId]
	   ,gui.[IsLeadership] AS IsLeadership
	   ,gui.[IsRecommended]
	   ,gui.CloseNote
	   ,s.[name] AS StatusName
	   ,'Guidance' AS RequestType
	   ,(SELECT
				COUNT(DISTINCT GE.FkOutsideEntityId)
			FROM Emergency.GuidanceEntity GE
			WHERE GE.FkGuidanceId = gui.Id)
		AS EntitiesCount
	   ,(SELECT
				COUNT(DISTINCT GE.FkOutsideEntityId)
			FROM Emergency.GuidanceEntityReply GER
			LEFT JOIN Emergency.GuidanceEntity GE
				ON GE.Id = GER.FkGuidanceEntityId
			WHERE GE.FkGuidanceId = gui.Id
			AND GER.FkStatusId = @DoneReplyStatus)
		AS RepliesCount
	   ,(SELECT
				COUNT(GER.Id)
			FROM Emergency.GuidanceEntityReply GER
			LEFT JOIN Emergency.GuidanceEntity GE
				ON GE.Id = GER.FkGuidanceEntityId
			WHERE GE.FkGuidanceId = gui.Id
			AND GER.FkStatusId = @WaitingReplyStatus
			AND (GE.FkOutsideEntityId = @EntityId
			OR @EntityId IS NULL))
		AS WaitingRepliesCount
		,(CASE
			WHEN EXISTS (
			SELECT 1
			FROM Emergency.GuidanceEntity GE
			WHERE GE.FkGuidanceId = gui.Id
			AND NOT EXISTS (
				SELECT 1
				FROM Emergency.GuidanceEntityReply GER
				WHERE GER.FkGuidanceEntityId = GE.Id
				AND GER.FkStatusId <> @DoneReplyStatus
			)
			)
			THEN 1
			ELSE 0
		  END
		)AS IsAllRepliesDone
		,(CASE
			WHEN (
				SELECT COUNT(*)
				FROM Emergency.GuidanceEntity GE
				WHERE GE.FkGuidanceId= gui.Id
			
			) = 1 THEN
				(
					SELECT TOP 1 OE.Name
					FROM Emergency.GuidanceEntity GE
					JOIN Lookup.OutsideEntity OE ON OE.Id = GE.FkOutsideEntityId
					WHERE GE.FkGuidanceId = gui.Id
				)
			WHEN (
				SELECT COUNT(*)
				FROM Emergency.GuidanceEntity GE
				WHERE GE.FkGuidanceId = gui.Id
			) > 1 THEN 'multiple'
			ELSE '0'
		 END
		) AS EntityName
	   ,ROW_NUMBER() OVER (ORDER BY gui.CreationDate DESC) AS RowNum
	FROM [Emergency].[Guidance] gui
	LEFT JOIN Login.UserProfile creationup
		ON gui.CreationUserId = creationup.Id
	LEFT JOIN Login.UserProfile lastupdateup
		ON gui.LastUpdateUserId = lastupdateup.Id
	LEFT JOIN Emergency.Incident inc
		ON gui.FkIncidentId = inc.Id
	LEFT JOIN Lookup.GuidanceType gt
		ON gui.FkGuidanceTypeId = gt.Id
	LEFT JOIN Lookup.status s
		ON gui.FkStatusId = s.id
	WHERE (inc.IncidentKey = @IncidentKey)
	AND (
	(inc.FkLeaderOutsideEntityId = @EntityId AND @IsLeadership = 1)
	OR ((@EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentLeadership
		WHERE FkIncidentId = inc.Id)
	)
	AND @IsLeadership = 1)
	OR (gui.FkLeadershipEntityId = @EntityId
	AND gui.IsLeadership = 0
	AND @IsLeadership = 0)
	OR (
	(gui.FkStatusId = @InformationPostedStatus OR gui.FkStatusId = @GuidanceDone OR gui.FkStatusId = @WaitingEntitiesStatus)
	AND @IsLeadership = 0
	or( @EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.GuidanceEntity
		WHERE FkGuidanceId = gui.Id)
	AND gui.FkStatusId <> @CloseStatus
	)
	)
	OR(gui.FkStatusId = @CloseStatus
	AND @IsLeadership = 0 
	AND @EntityId =gui.FkLeadershipEntityId
	)
	)
	UNION ALL
	SELECT
		inf.[Id]
	   ,inf.[InformationKey] AS RequestKey
	   ,inf.[InformationDate] AS RequestDate
	   ,inf.[Description]
	   ,inf.[FkInformationTypeId] AS TypeId
	   ,it.Name AS TypeName
	   ,inf.[FkIncidentId]
	   ,inf.[CreationDate]
	   ,inf.[CreationUserId]
	   ,inf.[LastUpdateDate]
	   ,inf.[LastUpdateUserId]
	   ,inf.[FkStatusId]
	   ,inf.[IsLeadership]
	   ,inf.[IsRecommended]
	   ,inf.CloseNote
	   ,s.[name] AS StatusName
	   ,'Information' AS RequestType
	   ,(SELECT
				COUNT(DISTINCT IE.FkOutsideEntityId)
			FROM Emergency.InformationEntity IE
			WHERE IE.FkInformationId = INF.Id)
		AS EntitiesCount
	   ,(SELECT
				COUNT(DISTINCT IE.FkOutsideEntityId)
			FROM Emergency.InformationEntityReply IER
			LEFT JOIN Emergency.InformationEntity IE
				ON IE.Id = IER.FkInformationEntityId
			WHERE IE.FkInformationId = INF.Id
			AND IER.FkStatusId = @DoneReplyStatus)
		AS RepliesCount
	   ,(SELECT
				COUNT(IER.Id)
			FROM Emergency.InformationEntityReply IER
			LEFT JOIN Emergency.InformationEntity IE
				ON IE.Id = IER.FkInformationEntityId
			WHERE IE.FkInformationId = INF.Id
			AND IER.FkStatusId = @WaitingReplyStatus
			AND (IE.FkOutsideEntityId = @EntityId
			OR @EntityId IS NULL))
		AS WaitingRepliesCount
		,(CASE
			WHEN EXISTS (
			SELECT 1
			FROM Emergency.InformationEntity IE
			WHERE IE.FkInformationId = inf.Id
			AND NOT EXISTS (
				SELECT 1
				FROM Emergency.InformationEntityReply IER
				WHERE IER.FkInformationEntityId = IE.Id
				AND IER.FkStatusId <> @DoneReplyStatus
			)
			)
			THEN 1
			ELSE 0
		  END
		) AS IsAllRepliesDone
		,(CASE
			WHEN (
				SELECT COUNT(*)
				FROM Emergency.InformationEntity IE
				WHERE IE.FkInformationId= inf.Id
			
			) = 1 THEN
				(
					SELECT TOP 1 OE.Name
					FROM Emergency.InformationEntity IE
					JOIN Lookup.OutsideEntity OE ON OE.Id = IE.FkOutsideEntityId
					WHERE IE.FkInformationId = inf.Id
				)
			WHEN (
				SELECT COUNT(*)
				FROM Emergency.InformationEntity IE
				WHERE IE.FkInformationId = inf.Id
			) > 1 THEN 'multiple'
			ELSE '0'
		 END
		) AS EntityName
	   ,ROW_NUMBER() OVER (ORDER BY inf.CreationDate DESC) AS RowNum
	FROM [Emergency].[Information] inf
	LEFT JOIN Login.UserProfile creationup
		ON inf.CreationUserId = creationup.Id
	LEFT JOIN Login.UserProfile lastupdateup
		ON inf.LastUpdateUserId = lastupdateup.Id
	LEFT JOIN Emergency.Incident inc
		ON inf.FkIncidentId = inc.Id
	LEFT JOIN Lookup.InformationType it
		ON inf.FkInformationTypeId = it.Id
	LEFT JOIN Lookup.status s
		ON inf.FkStatusId = s.id
	WHERE (inc.IncidentKey = @IncidentKey)
	AND (
	(inc.FkLeaderOutsideEntityId = @EntityId AND @IsLeadership = 1)
	OR (@EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentLeadership
		WHERE FkIncidentId = inc.Id)
	AND @IsLeadership = 1)
	OR (inf.FkLeadershipEntityId = @EntityId
	AND inf.IsLeadership = 0
	AND @IsLeadership = 0)
	OR (
	(inf.FkStatusId = @InformationPostedStatus OR inf.FkStatusId = @WaitingEntitiesStatus)
	AND @IsLeadership = 0
	or( @EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.InformationEntity
		WHERE FkInformationId = inf.Id)
	AND inf.FkStatusId <> @CloseStatus
	)
	)
	OR(inf.FkStatusId = @CloseStatus
	AND @IsLeadership = 0 
	AND @EntityId =inf.FkLeadershipEntityId
	)
	)) AS tab
END
