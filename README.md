(CASE
			WHEN EXISTS (
			SELECT 1
			FROM Emergency.InformationEntityReply IER
			LEFT JOIN Emergency.InformationEntity IE
				ON IE.Id = IER.FkInformationEntityId
			WHERE IE.FkInformationId = INF.Id 
			)AND NOT EXISTS (
			SELECT 1
			FROM Emergency.InformationEntityReply IER
			LEFT JOIN Emergency.InformationEntity IE
				ON IE.Id = IER.FkInformationEntityId
			WHERE IE.FkInformationId = INF.Id 
			AND IER.FkStatusId <> @DoneReplyStatus
			)
			THEN 1
			ELSE 0
		  END
		) AS IsAllRepliesDone
how to update this ti check if there is reply is not doe reply for each entity in information not for all replies of information
