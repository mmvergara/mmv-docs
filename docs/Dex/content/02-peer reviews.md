# Peer Reviews

- Every user can only create peer reviews ( but not for themselves )
- Admins can do CRUD on post's

## Client/Server Side Validation

Validating of peer reviews is hard especially on the evalution data part which will become JSONB data in the database, Both in the Server and Client, this project uses the combination of Yup and Formik to validate data

## Keyword Analysis

### Fetching

Using this database function

```sql
create or replace function public.employee_review_keyword_analysis(pattern text)
returns setof peer_reviews  as $$
  SELECT *
  FROM peer_reviews
  WHERE evaluation::text ~* pattern;
$$ language sql;
```

We can easily fetch the data we want to see base on the search query by invoking the function

```ts
const [reviews, setReviews] = useState<peer_reviews[] | null>(null);
const [searchQuery, setSearchQuery] = useState<string>("");
const pattern = searchQuery.split(",").join("|");

const fetchEmployeeReviews = async (supabase: SupabaseClient<DatabaseTypes>) => {
  if (pattern.length === 0) return;
  return (await supabase.rpc("employee_review_keyword_analysis", {
    pattern,
  })) as PostgrestResponse<peer_reviews>;
};
```

### Analyzation
Done in client side this function is responsible for it which will return an array of `key` `value` key being the search queries for ex. multiple queries can be done by seperating them by comman ( , )
```ts
export const employeeReviewKeywordAnalysis = (evaluations: peer_reviews[] | null, pattern: string) => {
  if (!pattern) return null;
  if (!evaluations) return null;

  const reviewKeywords = pattern.split("|");
  const keywordsAnalysis: KeywordsAnalysis = {};

  for (const evaluation of evaluations) {
    const seenKeyword = new Set<string>();
    const comments = getCommentsFromPR_required_ratings(evaluation.evaluation.required_rating);
    const optionalComments = Object.values(evaluation.evaluation.optional_rating).join(" ");
    const allComments = `${comments} ${optionalComments}`;

    for (const keyword of reviewKeywords) {
      if (seenKeyword.has(keyword)) continue;
      seenKeyword.add(keyword);

      if (allComments.includes(keyword)) {
        keywordsAnalysis[keyword] = {
          reviewsContainingKeyword: (keywordsAnalysis[keyword]?.reviewsContainingKeyword || 0) + 1,
          keywordOccurrences:
            (keywordsAnalysis[keyword]?.keywordOccurrences || 0) + countOccurrences(allComments, keyword),
        };
      } else {
        keywordsAnalysis[keyword] = {
          reviewsContainingKeyword: 0,
          keywordOccurrences: 0,
        };
      }
    }
  }

  return Object.entries(keywordsAnalysis);
};
```
Helpers

```ts
const getCommentsFromPR_required_ratings = (required_ratings: peer_review_required_ratings) => {
  const evaluationValues = Object.values(required_ratings);
  const comments = evaluationValues
    .map((ev) => {
      return ev.comment;
    })
    .join(" ");
  return comments;
};

type KeywordsAnalysis = {
  [key: string]: {
    keywordOccurrences: number;
    reviewsContainingKeyword: number;
  };
};
```