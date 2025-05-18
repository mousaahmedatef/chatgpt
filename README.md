USE [NCP_DEVTEAMDB]
GO
/****** Object:  StoredProcedure [dbo].[Emergency_Incident_GetAllPendingIncidentForEntity]    Script Date: 5/18/2025 3:02:15 PM ******/
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO
ALTER                        PROCEDURE [dbo].[Emergency_Incident_GetAllPendingIncidentForEntity]   
                                                                   @PageSize                                  INT,
                                                                   @PageNumber                                INT,
																   @OrderBy                                   NVARCHAR(250)  = N'CreationDate',
                                                                   @SortDir                                   NVARCHAR(10)   = 'DESC',
                                                                   @IncidentKey                                   NVARCHAR(250) = NULL,
                                                                   @EnteringName                                   NVARCHAR(250) = NULL,
                                                                   @FkEmergencyTypeId                                   INT = NULL,
                                                                   @FkEmergencyCategoryId                                   INT = NULL,
                                                                   @FkIncidentEnteringTypeId                                   INT = NULL,
                                                                   @FkCountryId                                   INT = NULL,
                                                                   @FkCityId                                   INT = NULL,
                                                                   @FkRegionId                                   INT = NULL,
                                                                   @FkAssigneeId                                   INT = NULL,
                                                                   @CreationDateFrom                                   DATETIME = NULL,
                                                                   @CreationDateTo                                   DATETIME = NULL,
                                                                   @RecommendNationalPlan                             NVARCHAR(250) = NULL,
																   @StatusString									  VARCHAR(max) = NULL,
																   @EntityId                                     INT = NULL,
																   @IsCivilDefense                                     BIT = NULL,
																   @BaseStatusString						     VARCHAR(max) = NULL,
																   @WaitingReplyStatus								INT,
																   @DoneReplyStatus									INT,
																   @ActivateNationalPlanStatus						INT,
																   @WaitingApprovalStatus							INT,
																   @WaitingPostStatus								INT,
																   @ForInformingStatus								INT,
																   @WaitingEditingStatus							INT,
																   @RecommendDeactivateStatus						INT,
																   @DeactivateNationalPlanStatus					INT,
																   @WaitingEntitiesStatus							INT,
																   @MainIncidents									BIT,
																   @CivilDefenseEntityId                            INT = NULL,
																   @LeadershipEntityId								INT = NULL
AS
BEGIN
     DECLARE @countrows DECIMAL;
-- Insert statements for procedure here
SELECT
	*
FROM (SELECT
		i.[Id]
	   ,i.[IncidentKey]
	   ,i.[FkStatusId]
	   ,s.[Name] AS StatusName
	   ,CASE
			WHEN oe.[Name] IS NOT NULL THEN oe.[Name]
			ELSE i.[EnteringName]
		END AS EnteringName
	   ,i.[FkIncidentEnteringTypeId]
	   ,iet.[Name] AS IncidentEnteringTypeName
	   ,i.[FkEmergencyTypeId]
	   ,et.[Name] AS EmergencyTypeName
	   ,i.[FkEmergencyCategoryId]
	   ,ec.[Name] AS EmergencyCategoryName
	   ,i.[EmergencyLocation]
	   ,i.[EmergencyDescription]
	   ,i.[EmergencyDate]
	   ,i.[IsOpened]
	   ,i.[IncidentDate]
	   ,i.[CreationDate]
	   ,i.[CreationUserId]
	   ,ISNULL(i.LastUpdateDate, i.CreationDate) AS LastUpdateDate
	   ,i.[LastUpdateUserId]
	   ,i.[EmergencyLink]
	   ,creationup.FullNameAr AS CreationUserName
	   ,lastupdateup.FullNameAr AS LastUpdateUserName
	   ,co.CountryKey AS CountryKey
	   ,co.Name AS CountryName
	   ,c.Name AS CityName
	   ,i.IsExternal
	   ,i.IsRead
	   ,CASE
			WHEN i.EmergencyLink IS NOT NULL THEN (CASE
					WHEN co.CountryKey = N'KSA' AND
						c.Name IS NOT NULL THEN c.Name
					ELSE (CASE
							WHEN co.Name IS NOT NULL THEN co.Name
							ELSE N'الرابط'
						END)
				END)
			ELSE (CASE
					WHEN co.CountryKey = N'KSA' AND
						c.Name IS NOT NULL THEN c.Name
					ELSE (CASE
							WHEN co.Name IS NOT NULL THEN co.Name
							ELSE N'-'
						END)
				END)
		END AS EmergencyLinkDesc
	   ,CASE
			WHEN (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus) THEN CASE
					WHEN @EntityId IS NOT NULL THEN ((SELECT
								COUNT(DISTINCT inf.Id)
							FROM Emergency.Information inf
							LEFT JOIN Emergency.InformationEntity infe
								ON infe.FkInformationId = inf.Id
							LEFT JOIN Emergency.InformationEntityReply infr
								ON infr.FkInformationEntityId = infe.Id
							WHERE inf.FkIncidentId = i.Id
							AND ((infr.FkStatusId = @WaitingReplyStatus
							AND infe.FkOutsideEntityId = @EntityId)
							OR (inf.FkStatusId = @WaitingEditingStatus
							AND inf.FkLeadershipEntityId = @EntityId
							AND inf.IsLeadership = 0)))
						+ (SELECT
								COUNT(DISTINCT gui.Id)
							FROM Emergency.Guidance gui
							LEFT JOIN Emergency.GuidanceEntity guie
								ON guie.FkGuidanceId = gui.Id
							LEFT JOIN Emergency.GuidanceEntityReply guir
								ON guir.FkGuidanceEntityId = guie.Id
							WHERE gui.FkIncidentId = i.Id
							AND ((guir.FkStatusId = @WaitingReplyStatus
							AND guie.FkOutsideEntityId = @EntityId)
							OR (gui.FkStatusId = @WaitingEditingStatus
							AND gui.FkLeadershipEntityId = @EntityId
							AND gui.IsLeadership = 0)))
						)
					WHEN @IsCivilDefense = 1 THEN ((SELECT
								COUNT(DISTINCT inf.Id)
							FROM Emergency.Information inf
							LEFT JOIN Emergency.InformationEntity infe
								ON infe.FkInformationId = inf.Id
							LEFT JOIN Emergency.InformationEntityReply infr
								ON infr.FkInformationEntityId = infe.Id
							WHERE inf.FkIncidentId = i.Id
							AND infr.FkStatusId = @WaitingReplyStatus
							AND infe.FkOutsideEntityId = @CivilDefenseEntityId)
						+ (SELECT
								COUNT(DISTINCT gui.Id)
							FROM Emergency.Guidance gui
							LEFT JOIN Emergency.GuidanceEntity guie
								ON guie.FkGuidanceId = gui.Id
							LEFT JOIN Emergency.GuidanceEntityReply guir
								ON guir.FkGuidanceEntityId = guie.Id
							WHERE gui.FkIncidentId = i.Id
							AND guir.FkStatusId = @WaitingReplyStatus
							AND guie.FkOutsideEntityId = @CivilDefenseEntityId)
						)
					WHEN @LeadershipEntityId IS NOT NULL AND
						i.FkLeaderOutsideEntityId = @LeadershipEntityId THEN ((SELECT
								COUNT(DISTINCT inf.Id)
							FROM Emergency.Information inf
							LEFT JOIN Emergency.InformationEntity infe
								ON infe.FkInformationId = inf.Id
							LEFT JOIN Emergency.InformationEntityReply infr
								ON infr.FkInformationEntityId = infe.Id
							WHERE inf.FkIncidentId = i.Id
							AND ((infr.FkStatusId = @DoneReplyStatus AND inf.FkStatusId = @WaitingEntitiesStatus)
							OR inf.FkStatusId = @WaitingApprovalStatus
							OR inf.FkStatusId = @WaitingPostStatus))
						+ (SELECT
								COUNT(DISTINCT gui.Id)
							FROM Emergency.Guidance gui
							LEFT JOIN Emergency.GuidanceEntity guie
								ON guie.FkGuidanceId = gui.Id
							LEFT JOIN Emergency.GuidanceEntityReply guir
								ON guir.FkGuidanceEntityId = guie.Id
							WHERE gui.FkIncidentId = i.Id
							AND ((guir.FkStatusId = @DoneReplyStatus AND gui.FkStatusId = @WaitingEntitiesStatus)
							OR gui.FkStatusId = @WaitingApprovalStatus
							OR gui.FkStatusId = @WaitingPostStatus))
						+ (SELECT
								COUNT(DISTINCT Id)
							FROM Emergency.IncidentCloseRecommendation
							WHERE FkIncidentId = i.Id
							AND FkStatusId = @RecommendDeactivateStatus)
						)
					WHEN @LeadershipEntityId IS NOT NULL AND
						(@LeadershipEntityId IN (SELECT
								FkOutsideEntityId
							FROM Emergency.IncidentLeadership
							WHERE FkIncidentId = i.Id)
						) THEN ((SELECT
								COUNT(DISTINCT inf.Id)
							FROM Emergency.Information inf
							WHERE inf.FkIncidentId = i.Id
							AND inf.FkStatusId = @WaitingEditingStatus
							AND inf.FkLeadershipEntityId = @LeadershipEntityId
							AND inf.isLeadership = 1)
						+ (SELECT
								COUNT(DISTINCT gui.Id)
							FROM Emergency.Guidance gui
							WHERE gui.FkIncidentId = i.Id
							AND gui.FkStatusId = @WaitingEditingStatus
							AND gui.FkLeadershipEntityId = @LeadershipEntityId
							AND gui.IsLeadership = 1)
						+ (SELECT
								COUNT(DISTINCT Id)
							FROM Emergency.IncidentCloseRecommendation
							WHERE FkIncidentId = i.Id
							AND FkStatusId = @WaitingEditingStatus
							AND FkOutsideEntityId = @LeadershipEntityId)
						)
					ELSE 0
				END
			ELSE 0
		END AS ActionsCount
	FROM Emergency.Incident i
	LEFT JOIN Login.UserProfile creationup
		ON i.CreationUserId = creationup.Id
	LEFT JOIN Login.UserProfile lastupdateup
		ON i.LastUpdateUserId = lastupdateup.Id
	LEFT JOIN Lookup.IncidentEnteringType iet
		ON i.FkIncidentEnteringTypeId = iet.Id
	LEFT JOIN Lookup.EmergencyType et
		ON i.FkEmergencyTypeId = et.Id
	LEFT JOIN Lookup.EmergencyCategory ec
		ON i.FkEmergencyCategoryId = ec.Id
	LEFT JOIN Lookup.Status s
		ON i.FkStatusId = s.Id
	LEFT JOIN Lookup.City c
		ON i.fkCityId = c.Id
	LEFT JOIN Lookup.Country co
		ON i.FKCOuntryId = co.Id
	LEFT JOIN Lookup.OutsideEntity oe
		ON i.FkEnteringOutsideEntityId = oe.Id
	WHERE (i.FkEmergencyTypeId = @FkEmergencyTypeId
	OR @FkEmergencyTypeId IS NULL)
	AND (i.FkStatusId IN (SELECT
			*
		FROM [dbo].[splitstring](@StatusString))
	OR @StatusString IS NULL)
	AND (i.FkEmergencyCategoryId = @FkEmergencyCategoryId
	OR @FkEmergencyCategoryId IS NULL)
	AND (i.FkIncidentEnteringTypeId = @FkIncidentEnteringTypeId
	OR @FkIncidentEnteringTypeId IS NULL)
	AND (i.FkCountryId = @FkCountryId
	OR @FkCountryId IS NULL)
	AND (i.FkCityId = @FkCityId
	OR @FkCityId IS NULL)
	AND (i.FkRegionId = @FkRegionId
	OR @FkRegionId IS NULL)
	AND (i.FkAssigneeId = @FkAssigneeId
	OR @FkAssigneeId IS NULL)
	AND (i.RecommendNationalPlan = @RecommendNationalPlan
	OR @RecommendNationalPlan IS NULL)
	AND (i.IncidentKey LIKE '%' + @IncidentKey + '%'
	OR i.IncidentMasterKey LIKE '%' + @IncidentKey + '%'
	OR i.IncidentKey IN (SELECT
			IncidentMasterKey
		FROM Emergency.Incident
		WHERE IncidentKey LIKE '%' + @IncidentKey + '%')
	OR i.IncidentMasterKey IN (SELECT
			IncidentMasterKey
		FROM Emergency.Incident
		WHERE IncidentKey LIKE '%' + @IncidentKey + '%')
	OR @IncidentKey IS NULL)
	AND (i.EnteringName LIKE '%' + @EnteringName + '%'
	OR @EnteringName IS NULL)
	AND (CAST(i.IncidentDate AS DATE) >= CAST(@CreationDateFrom AS DATE)
	OR @CreationDateFrom IS NULL)
	AND (CAST(i.IncidentDate AS DATE) <= CAST(@CreationDateTo AS DATE)
	OR @CreationDateTo IS NULL)
	AND (
	(i.FkStatusId = @ForInformingStatus
	AND @EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentEntity
		WHERE FkIncidentId = i.Id
		AND ExternalIsRead = 0)
	)
	OR ((@EntityId IN (SELECT
			infe.FkOutsideEntityId
		FROM Emergency.Information inf
		LEFT JOIN Emergency.InformationEntity infe
			ON infe.FkInformationId = inf.Id
		LEFT JOIN Emergency.InformationEntityReply infr
			ON infr.FkInformationEntityId = infe.Id
		WHERE inf.FkIncidentId = i.Id
		AND (infr.FkStatusId = @WaitingReplyStatus
		OR (inf.FkStatusId = @WaitingEditingStatus
		AND inf.FkLeadershipEntityId = @EntityId
		AND inf.IsLeadership = 0)))
	OR @EntityId IN (SELECT
			guie.FkOutsideEntityId
		FROM Emergency.Guidance gui
		LEFT JOIN Emergency.GuidanceEntity guie
			ON guie.FkGuidanceId = gui.Id
		LEFT JOIN Emergency.GuidanceEntityReply guir
			ON guir.FkGuidanceEntityId = guie.Id
		WHERE gui.FkIncidentId = i.Id
		AND (guir.FkStatusId = @WaitingReplyStatus
		OR (gui.FkStatusId = @WaitingEditingStatus
		AND gui.FkLeadershipEntityId = @EntityId
		AND gui.IsLeadership = 0)))
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus))
	OR @EntityId IS NULL)
	AND (
	(i.FkStatusId IN (SELECT
			*
		FROM [dbo].[splitstring](@BaseStatusString))
	)
	OR (i.FkStatusId = @ForInformingStatus
	AND i.IsRead = 0
	AND NOT EXISTS (SELECT
			Id
		FROM Emergency.IncidentEntity
		WHERE FkIncidentId = i.Id)
	)
	OR (i.FkStatusId = @ForInformingStatus
	AND @CivilDefenseEntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentEntity
		WHERE FkIncidentId = i.Id
		AND ExternalIsRead = 0)
	)
	OR (((@CivilDefenseEntityId IN (SELECT
			infe.FkOutsideEntityId
		FROM Emergency.Information inf
		LEFT JOIN Emergency.InformationEntity infe
			ON infe.FkInformationId = inf.Id
		LEFT JOIN Emergency.InformationEntityReply infr
			ON infr.FkInformationEntityId = infe.Id
		WHERE inf.FkIncidentId = i.Id
		AND infr.FkStatusId = @WaitingReplyStatus)
	OR @CivilDefenseEntityId IN (SELECT
			guie.FkOutsideEntityId
		FROM Emergency.Guidance gui
		LEFT JOIN Emergency.GuidanceEntity guie
			ON guie.FkGuidanceId = gui.Id
		LEFT JOIN Emergency.GuidanceEntityReply guir
			ON guir.FkGuidanceEntityId = guie.Id
		WHERE gui.FkIncidentId = i.Id
		AND guir.FkStatusId = @WaitingReplyStatus)
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus)))
	OR @IsCivilDefense IS NULL)
	AND (
	(i.FkLeaderOutsideEntityId = @LeadershipEntityId
	AND (
	(EXISTS (SELECT
			infe.FkOutsideEntityId
		FROM Emergency.Information inf
		LEFT JOIN Emergency.InformationEntity infe
			ON infe.FkInformationId = inf.Id
		LEFT JOIN Emergency.InformationEntityReply infr
			ON infr.FkInformationEntityId = infe.Id
		WHERE inf.FkIncidentId = i.Id
		AND ((infr.FkStatusId = @DoneReplyStatus AND inf.FkStatusId = @WaitingEntitiesStatus)
		OR inf.FkStatusId = @WaitingApprovalStatus
		OR inf.FkStatusId = @WaitingPostStatus))
	OR EXISTS (SELECT
			guie.FkOutsideEntityId
		FROM Emergency.Guidance gui
		LEFT JOIN Emergency.GuidanceEntity guie
			ON guie.FkGuidanceId = gui.Id
		LEFT JOIN Emergency.GuidanceEntityReply guir
			ON guir.FkGuidanceEntityId = guie.Id
		WHERE gui.FkIncidentId = i.Id
		AND ((guir.FkStatusId = @DoneReplyStatus AND gui.FkStatusId = @WaitingEntitiesStatus)
		OR gui.FkStatusId = @WaitingApprovalStatus
		OR gui.FkStatusId = @WaitingPostStatus))
	OR EXISTS (SELECT
			Id
		FROM Emergency.IncidentCloseRecommendation
		WHERE FkIncidentId = i.Id
		AND FkStatusId = @RecommendDeactivateStatus)
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus))
	)
	OR (@LeadershipEntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentLeadership
		WHERE FkIncidentId = i.Id)
	AND (
	((
	EXISTS (SELECT
			inf.Id
		FROM Emergency.Information inf
		WHERE inf.FkIncidentId = i.Id
		AND inf.FkStatusId = @WaitingEditingStatus
		AND inf.FkLeadershipEntityId = @LeadershipEntityId
		AND inf.isLeadership = 1)
	OR EXISTS (SELECT
			gui.Id
		FROM Emergency.Guidance gui
		WHERE gui.FkIncidentId = i.Id
		AND gui.FkStatusId = @WaitingEditingStatus
		AND gui.FkLeadershipEntityId = @LeadershipEntityId
		AND gui.IsLeadership = 1)
	OR EXISTS (SELECT
			Id
		FROM Emergency.IncidentCloseRecommendation
		WHERE FkIncidentId = i.Id
		AND FkStatusId = @WaitingEditingStatus
		AND FkOutsideEntityId = @LeadershipEntityId)
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus))
	)
	)
	OR @LeadershipEntityId IS NULL
	)
	AND (i.IncidentMasterKey IS NULL
	OR @MainIncidents = 0)) AS tab
ORDER BY CASE
	WHEN tab.FkStatusId = @ActivateNationalPlanStatus AND
		@OrderBy = N'CreationDate' THEN 0
	ELSE 1
END ASC,
CASE
	WHEN @OrderBy = N'IncidentKey' AND
		@SortDir = 'ASC' THEN tab.CreationDate
END ASC,
CASE
	WHEN @OrderBy = N'IncidentKey' AND
		@SortDir = 'DESC' THEN tab.CreationDate
END DESC,
CASE
	WHEN @OrderBy = N'CreationDate' AND
		@SortDir = 'ASC' THEN tab.CreationDate
END ASC,
CASE
	WHEN @OrderBy = N'CreationDate' AND
		@SortDir = 'DESC' THEN tab.CreationDate
END DESC,
CASE
	WHEN @OrderBy = N'IncidentDate' AND
		@SortDir = 'ASC' THEN tab.IncidentDate
END ASC,
CASE
	WHEN @OrderBy = N'IncidentDate' AND
		@SortDir = 'DESC' THEN tab.IncidentDate
END DESC,
CASE
	WHEN @OrderBy = N'EnteringName' AND
		@SortDir = 'ASC' THEN tab.EnteringName
END ASC,
CASE
	WHEN @OrderBy = N'EnteringName' AND
		@SortDir = 'DESC' THEN tab.EnteringName
END DESC,
CASE
	WHEN @OrderBy = N'EmergencyDate' AND
		@SortDir = 'ASC' THEN tab.EmergencyDate
END ASC,
CASE
	WHEN @OrderBy = N'EmergencyDate' AND
		@SortDir = 'DESC' THEN tab.EmergencyDate
END DESC,
CASE
	WHEN @OrderBy = N'IncidentEnteringTypeName' AND
		@SortDir = 'ASC' THEN tab.IncidentEnteringTypeName
END ASC,
CASE
	WHEN @OrderBy = N'IncidentEnteringTypeName' AND
		@SortDir = 'DESC' THEN tab.IncidentEnteringTypeName
END DESC,
CASE
	WHEN @OrderBy = N'EmergencyTypeName' AND
		@SortDir = 'ASC' THEN tab.EmergencyTypeName
END ASC,
CASE
	WHEN @OrderBy = N'EmergencyTypeName' AND
		@SortDir = 'DESC' THEN tab.EmergencyTypeName
END DESC,
CASE
	WHEN @OrderBy = N'EmergencyCategoryName' AND
		@SortDir = 'ASC' THEN tab.EmergencyCategoryName
END ASC,
CASE
	WHEN @OrderBy = N'EmergencyCategoryName' AND
		@SortDir = 'DESC' THEN tab.EmergencyCategoryName
END DESC,
CASE
	WHEN @OrderBy = N'StatusName' AND
		@SortDir = 'ASC' THEN tab.StatusName
END ASC,
CASE
	WHEN @OrderBy = N'StatusName' AND
		@SortDir = 'DESC' THEN tab.StatusName
END DESC,
CASE
	WHEN @OrderBy = N'EmergencyLinkDesc' AND
		@SortDir = 'ASC' THEN tab.EmergencyLinkDesc
END ASC,
CASE
	WHEN @OrderBy = N'EmergencyLinkDesc' AND
		@SortDir = 'DESC' THEN tab.EmergencyLinkDesc
END DESC,
CASE
	WHEN @OrderBy = N'LastUpdateDate' AND
		@SortDir = 'ASC' THEN tab.LastUpdateDate
END ASC,
CASE
	WHEN @OrderBy = N'LastUpdateDate' AND
		@SortDir = 'DESC' THEN tab.LastUpdateDate
END DESC
OFFSET (@PageNumber - 1) * (@PageSize) ROWS FETCH NEXT (@PageSize) ROWS ONLY;
SET @countrows = (SELECT
		COUNT(i.Id)
	FROM Emergency.Incident i
	LEFT JOIN Login.UserProfile creationup
		ON i.CreationUserId = creationup.Id
	LEFT JOIN Login.UserProfile lastupdateup
		ON i.LastUpdateUserId = lastupdateup.Id
	LEFT JOIN Lookup.IncidentEnteringType iet
		ON i.FkIncidentEnteringTypeId = iet.Id
	LEFT JOIN Lookup.EmergencyType et
		ON i.FkEmergencyTypeId = et.Id
	LEFT JOIN Lookup.EmergencyCategory ec
		ON i.FkEmergencyCategoryId = ec.Id
	LEFT JOIN Lookup.Status s
		ON i.FkStatusId = s.Id
	LEFT JOIN Lookup.City c
		ON i.fkCityId = c.Id
	LEFT JOIN Lookup.Country co
		ON i.FKCOuntryId = co.Id
	LEFT JOIN Lookup.OutsideEntity oe
		ON i.FkEnteringOutsideEntityId = oe.Id
	WHERE (i.FkEmergencyTypeId = @FkEmergencyTypeId
	OR @FkEmergencyTypeId IS NULL)
	AND (i.FkStatusId IN (SELECT
			*
		FROM [dbo].[splitstring](@StatusString))
	OR @StatusString IS NULL)
	AND (i.FkEmergencyCategoryId = @FkEmergencyCategoryId
	OR @FkEmergencyCategoryId IS NULL)
	AND (i.FkIncidentEnteringTypeId = @FkIncidentEnteringTypeId
	OR @FkIncidentEnteringTypeId IS NULL)
	AND (i.FkCountryId = @FkCountryId
	OR @FkCountryId IS NULL)
	AND (i.FkCityId = @FkCityId
	OR @FkCityId IS NULL)
	AND (i.FkRegionId = @FkRegionId
	OR @FkRegionId IS NULL)
	AND (i.FkAssigneeId = @FkAssigneeId
	OR @FkAssigneeId IS NULL)
	AND (i.RecommendNationalPlan = @RecommendNationalPlan
	OR @RecommendNationalPlan IS NULL)
	AND (i.IncidentKey LIKE '%' + @IncidentKey + '%'
	OR i.IncidentMasterKey LIKE '%' + @IncidentKey + '%'
	OR i.IncidentKey IN (SELECT
			IncidentMasterKey
		FROM Emergency.Incident
		WHERE IncidentKey LIKE '%' + @IncidentKey + '%')
	OR i.IncidentMasterKey IN (SELECT
			IncidentMasterKey
		FROM Emergency.Incident
		WHERE IncidentKey LIKE '%' + @IncidentKey + '%')
	OR @IncidentKey IS NULL)
	AND (i.EnteringName LIKE '%' + @EnteringName + '%'
	OR @EnteringName IS NULL)
	AND (CAST(i.IncidentDate AS DATE) >= CAST(@CreationDateFrom AS DATE)
	OR @CreationDateFrom IS NULL)
	AND (CAST(i.IncidentDate AS DATE) <= CAST(@CreationDateTo AS DATE)
	OR @CreationDateTo IS NULL)
	AND (
	(i.FkStatusId = @ForInformingStatus
	AND @EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentEntity
		WHERE FkIncidentId = i.Id
		AND ExternalIsRead = 0)
	)
	OR ((@EntityId IN (SELECT
			infe.FkOutsideEntityId
		FROM Emergency.Information inf
		LEFT JOIN Emergency.InformationEntity infe
			ON infe.FkInformationId = inf.Id
		LEFT JOIN Emergency.InformationEntityReply infr
			ON infr.FkInformationEntityId = infe.Id
		WHERE inf.FkIncidentId = i.Id
		AND (infr.FkStatusId = @WaitingReplyStatus
		OR (inf.FkStatusId = @WaitingEditingStatus
		AND inf.FkLeadershipEntityId = @EntityId
		AND inf.IsLeadership = 0)))
	OR @EntityId IN (SELECT
			guie.FkOutsideEntityId
		FROM Emergency.Guidance gui
		LEFT JOIN Emergency.GuidanceEntity guie
			ON guie.FkGuidanceId = gui.Id
		LEFT JOIN Emergency.GuidanceEntityReply guir
			ON guir.FkGuidanceEntityId = guie.Id
		WHERE gui.FkIncidentId = i.Id
		AND (guir.FkStatusId = @WaitingReplyStatus
		OR (gui.FkStatusId = @WaitingEditingStatus
		AND gui.FkLeadershipEntityId = @EntityId
		AND gui.IsLeadership = 0)))
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus))
	OR @EntityId IS NULL)
	AND (
	(i.FkStatusId IN (SELECT
			*
		FROM [dbo].[splitstring](@BaseStatusString))
	)
	OR (i.FkStatusId = @ForInformingStatus
	AND i.IsRead = 0
	AND NOT EXISTS (SELECT
			Id
		FROM Emergency.IncidentEntity
		WHERE FkIncidentId = i.Id)
	)
	OR (i.FkStatusId = @ForInformingStatus
	AND @CivilDefenseEntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentEntity
		WHERE FkIncidentId = i.Id
		AND ExternalIsRead = 0)
	)
	OR (((@CivilDefenseEntityId IN (SELECT
			infe.FkOutsideEntityId
		FROM Emergency.Information inf
		LEFT JOIN Emergency.InformationEntity infe
			ON infe.FkInformationId = inf.Id
		LEFT JOIN Emergency.InformationEntityReply infr
			ON infr.FkInformationEntityId = infe.Id
		WHERE inf.FkIncidentId = i.Id
		AND infr.FkStatusId = @WaitingReplyStatus)
	OR @CivilDefenseEntityId IN (SELECT
			guie.FkOutsideEntityId
		FROM Emergency.Guidance gui
		LEFT JOIN Emergency.GuidanceEntity guie
			ON guie.FkGuidanceId = gui.Id
		LEFT JOIN Emergency.GuidanceEntityReply guir
			ON guir.FkGuidanceEntityId = guie.Id
		WHERE gui.FkIncidentId = i.Id
		AND guir.FkStatusId = @WaitingReplyStatus)
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus)))
	OR @IsCivilDefense IS NULL)
	AND (
	(i.FkLeaderOutsideEntityId = @LeadershipEntityId
	AND (
	(EXISTS (SELECT
			infe.FkOutsideEntityId
		FROM Emergency.Information inf
		LEFT JOIN Emergency.InformationEntity infe
			ON infe.FkInformationId = inf.Id
		LEFT JOIN Emergency.InformationEntityReply infr
			ON infr.FkInformationEntityId = infe.Id
		WHERE inf.FkIncidentId = i.Id
		AND ((infr.FkStatusId = @DoneReplyStatus AND inf.FkStatusId = @WaitingEntitiesStatus)
		OR inf.FkStatusId = @WaitingApprovalStatus
		OR inf.FkStatusId = @WaitingPostStatus))
	OR EXISTS (SELECT
			guie.FkOutsideEntityId
		FROM Emergency.Guidance gui
		LEFT JOIN Emergency.GuidanceEntity guie
			ON guie.FkGuidanceId = gui.Id
		LEFT JOIN Emergency.GuidanceEntityReply guir
			ON guir.FkGuidanceEntityId = guie.Id
		WHERE gui.FkIncidentId = i.Id
		AND ((guir.FkStatusId = @DoneReplyStatus AND gui.FkStatusId = @WaitingEntitiesStatus)
		OR gui.FkStatusId = @WaitingApprovalStatus
		OR gui.FkStatusId = @WaitingPostStatus))
	OR EXISTS (SELECT
			Id
		FROM Emergency.IncidentCloseRecommendation
		WHERE FkIncidentId = i.Id
		AND FkStatusId = @RecommendDeactivateStatus)
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus))
	)
	OR (@LeadershipEntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.IncidentLeadership
		WHERE FkIncidentId = i.Id)
	AND (
	((
	EXISTS (SELECT
			inf.Id
		FROM Emergency.Information inf
		WHERE inf.FkIncidentId = i.Id
		AND inf.FkStatusId = @WaitingEditingStatus
		AND inf.FkLeadershipEntityId = @LeadershipEntityId
		AND inf.isLeadership = 1)
	OR EXISTS (SELECT
			gui.Id
		FROM Emergency.Guidance gui
		WHERE gui.FkIncidentId = i.Id
		AND gui.FkStatusId = @WaitingEditingStatus
		AND gui.FkLeadershipEntityId = @LeadershipEntityId
		AND gui.IsLeadership = 1)
	OR EXISTS (SELECT
			Id
		FROM Emergency.IncidentCloseRecommendation
		WHERE FkIncidentId = i.Id
		AND FkStatusId = @WaitingEditingStatus
		AND FkOutsideEntityId = @LeadershipEntityId)
	)
	AND (i.FkStatusId = @ActivateNationalPlanStatus OR i.FkStatusId = @DeactivateNationalPlanStatus))
	)
	)
	OR @LeadershipEntityId IS NULL
	)
	AND (i.IncidentMasterKey IS NULL
	OR @MainIncidents = 0));
SELECT
	CEILING(@countrows / @PageSize) AS TotalPages
   ,@PageNumber AS CurrentPage
   ,@PageSize AS PageSize
   ,@countrows AS TotalRows
END
apply this also in WHEN @LeadershipEntityId IS NOT NULL AND section to check if all is done
