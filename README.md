private async Task<IEnumerable<PollVote>> PollVotesWithOutSideEntityName(IEnumerable<PollVote> pollVotes )
{
    var pollVotesIds = pollVotes.Select(i => i.Id).ToList();

    var outsideEntities = await _outsideEntity.FindByConditionWithSelect(i => pollVotesIds.Contains(i.Id), i => new KeyValuePair <int,string> ( i.Id, i.Name ));

    foreach (var pollVote in pollVotes)
    {
        var outsideEntity = outsideEntities.FirstOrDefault(e => e.Id ==  pollVote.FkOutsideEntityId);
        if (outsideEntity != null)
        {
            pollVote.OutsideEntity.Name = outsideEntity.Name;
        }
    }

    return pollVotes;
}
