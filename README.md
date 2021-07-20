# golang

Query database 

type History struct {
	ID          primitive.ObjectID `bson:"_id,omitempty" json:"_id" form:"-" query:"_id"`
	Users       Users              `bson:"user_id,omitempty" json:"user_id" form:"-" query:"user_id"`
	Ip_address  string             `bson:"ip_address,omitempty" json:"ip_address" form:"-" query:"ip_address"`
	Device      string             `bson:"device,omitempty" json:"device" form:"-" query:"device"`
	Last_update string             `bson:"last_updated,omitempty" json:"last_updated" form:"-" query:"last_updated"`
}

func (h *History) GetLoginHistory() ([]map[string]interface{}, error) {
	collection := database.MongoClient.Database(keys.Database).Collection("login_histories")
	aggregateStage := []bson.M{
		
		{
			"$lookup": bson.M{
				"from":         "users",
				"localField":   "user_id",
				"foreignField": "_id",
				"as":           "users",
			},
		},
		{
			"$unwind": bson.M{
				"path":                       "$users",
				"preserveNullAndEmptyArrays": true,
			},
		},
		{"$sort": bson.M{"created_date": -1}},
		{
			"$project": bson.M{
				"_id":          1,
				"ip_address":   1,
				"device":       1,
				"last_updated": 1,
				"users": bson.M{
					"_id":        1,
					"image":      1,
					"first_name": 1,
					"last_name":  1,
					"groups":     1,
					"staff_id":   1,
				},
			},
		},
	}

	cursor, err := collection.Aggregate(ctx, aggregateStage)
	if err != nil {
		fmt.Println(err)
		return nil, err
	}
	result := []map[string]interface{}{}
	if err := cursor.All(ctx, &result); err != nil {
		fmt.Println(err)
		return nil, err
	}
	return result, nil
}
