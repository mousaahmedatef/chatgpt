  [HttpPost]
  public virtual async Task<IActionResult> Post([FromBody] TDto dto)
  {
      if (dto == null)
      { 
          return BadRequest("DTO cannot be null");
      }

      var createdDto = await _service.AddAsync(dto);
      
      return Ok(createdDto);
  }

  are this method body type is json or From-Data in .net
