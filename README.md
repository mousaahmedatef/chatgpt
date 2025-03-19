public async Task<IEnumerable<object>> FindByConditionWithSelect(Expression<Func<TEntity, bool>> expression, Expression<Func<TEntity, object>> selectExpression)
{
    return await context.Set<TEntity>().AsNoTracking().Where(expression).Select(selectExpression).ToListAsync();
}
 
 public async Task<IEnumerable<PollVote>> GetPollVotesForIncidentPoll(int incidentPollId)
 {
     var pollVotes= await _pollVoteRopository.GetWithIncludesAndCondition(i => i.FkIncidentPollId == incidentPollId, i => i.OutsideEntity).;
     var pollVotesIds = pollVotes.Select(i => i.Id).ToList();

     var outsideEntities = await _outsideEntity.FindByConditionWithSelect(i => pollVotesIds.Contains(i.Id), i => { i.Id , i.Name});
 }
