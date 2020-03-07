#!/bin/sh

# get full path to file
path=$(realpath $1)

# get title      vvvvvvvvvvvv only first occurrence
title=$(sed -En '0,/^title: / s/^title: (.*)$/\1/p' < $path)

# get raw list of tags
tags_unsorted=($(sed -En '0,/^tags: / s/^tags: (.*)$/\1/p' < $path))

# check if tags exist
if [[ -n $tags_unsorted ]]; then
	# sort 'em
	tags=($(for i in "${tags_unsorted[@]}"; do echo $i; done | sort))
	# format them as {"tag1", "tag2", ..., "tagn"}
	tag_string="{"
	for i in ${!tags[@]}
	do
		tag=${tags[$i]}
		tag_string="${tag_string}\"${tag}\""
		[[ $i -lt $((${#tags[@]} - 1)) ]] && tag_string="${tag_string}, "
	done
	tag_string="${tag_string}}"
fi

# holy SQL injection, batman!
# if the user wants to SQL inject their own zettelkasten,
# quite honestly, that's their prerogative
# form SQL command string
if [[ -n $tag_string ]]; then
	psql_query="INSERT INTO note (title, tags, file)
	            VALUES ('${title}', '${tag_string}', '${path}')
				RETURNING id;"
else
	psql_query="INSERT INTO note (title, file)
	            VALUES ('${title}', '${path}')
				RETURNING id;"
fi

echo $psql_query | psql -d zettelkasten --no-align --field-separator="" --quiet --tuples-only
