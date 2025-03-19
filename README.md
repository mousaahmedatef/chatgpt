public async Task<IEnumerable<KeyValuePair<int, string>>> FindByConditionWithSelect(Expression<Func<TEntity, bool>> expression, Expression<Func<TEntity, object>> selectExpression)
{
    return await context.Set<TEntity>().AsNoTracking().Where(expression).Select(selectExpression).ToListAsync();
}

update tis to return key value pair
