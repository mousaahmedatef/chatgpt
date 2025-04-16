 public async Task<IEnumerable<TEntity>> GetWithIncludesAndCondition(Expression<Func<TEntity, bool>>? condition = null, params Expression<Func<TEntity, object>>[] includes)
 {
     IQueryable<TEntity> query = context.Set<TEntity>();
     foreach (var include in includes)
     {
         query = query.Include(include);
     }
     if (condition is not null)
         return await query.Where(condition).ToListAsync();
     else
         return await query.ToListAsync();
 }
