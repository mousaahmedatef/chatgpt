Severity	Code	Description	Project	File	Line	Suppression State
Error	CS1929	'IEnumerable<Incident>' does not contain a definition for 'Contains' and the best extension method overload 'MemoryExtensions.Contains<int>(ReadOnlySpan<int>, int)' requires a receiver of type 'System.ReadOnlySpan<int>'	Services	C:\Users\yhassan\source\repos\NRRC-Emergency_Backend\Services\Admin\Emergency\IncidentPoll\IncidentPollService.cs	117	Active

for this lines
 var incidentsForCurrentEnttiy = await _incidentRepository.FindByCondition(i => i.IncidentLeaderships.Any(il => il.FkOutsideEntityId == entityId)&& i.IncidentPolls.Any());
 var polls = await _repository.FindByCondition(p => !p.PollVotes.Any(v => v.FkOutsideEntityId == entityId) && incidentsForCurrentEnttiy.Contains(p.FkIncidentId));


