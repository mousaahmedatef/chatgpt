_repository.FindByCondition(p => !p.PollVotes.Any(v => v.FkOutsideEntityId == entityId) && p.FkIncidentId == fkIncidentId && p.FkStatusId != (int)IncidentPollStatusEnum.CLOSED);
can you change this to sel server querey
