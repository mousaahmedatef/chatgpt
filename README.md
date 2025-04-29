or @EntityId IN (SELECT
			FkOutsideEntityId
		FROM Emergency.InformationEntity
		WHERE FkInformationId = inf.Id)
	)
