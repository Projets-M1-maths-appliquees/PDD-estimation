# Provision pour Dépréciation Durable (PDD) — Simulation et Analyse

## Présentation du projet

Ce projet implémente en Python une estimation de la **Provision pour Dépréciation Durable (PDD)** d’actifs financiers. La PDD est une réserve comptable qui anticipe les pertes **durables** (non temporaires) d’actifs, par exemple lorsqu’un titre reste sous 75% de sa valeur d’acquisition pendant 6 mois consécutifs.

Nous simulons des trajectoires d’actifs sous dynamique stochastique (Black-Scholes), et nous estimons la PDD via des méthodes **Monte Carlo**. Deux techniques sont comparées :  
- Une **méthode naïve** par discrétisation,  
- Une **méthode raffinée** utilisant un **pont brownien**.

---

## Modélisation mathématique

### Dynamique des actifs

Les actifs $S_t$ sont modélisés sous la mesure risque-neutre $\mathbb{Q}$ par une équation de Black-Scholes :

$$
dS_t = r S_t dt + \sigma S_t dW_t
$$

où :
- $r$ est le taux sans risque,
- $\sigma$ est la volatilité,
- $W_t$ est un mouvement brownien standard.

La solution exacte de cette EDS est donnée par la formule fermée :

$$
S_t = S_0 \cdot \exp \left( \left( r - \frac{1}{2} \sigma^2 \right) t + \sigma W_t \right)
$$

---

### PDD espéré sur une intervalle de temps prolongée

Pour un actif acquis au prix $S_a$, la PDD est calculée lorsqu’une **double condition** est remplie :  
1. En fin de période $t+1$ : $S_{t+1} \leq \alpha S_a$ (baisse d’au moins $\alpha$).  
2. Pendant le dernier semestre de la période :  

$$
\sup_{u \in (t + 1/2, t+1]} S_u \leq S_a
$$

Cela garantit une **dépréciation durable**.

La perte conditionnelle $\lambda_t$ est alors définie par :

$$ \lambda_t = (S_a - S_{t+1})^+ \cdot \mathbf{1}_{\{ S_{t+1} \leq \alpha S_a \}} $$


$$
\lambda_t(r, \sigma, S_a, S_0) = \left( S_a - S_{t+1} \right)^+ \cdot \mathbf{1}\left\{ S_{t+1} \leq \alpha S_a \right\} \cdot \mathbf{1} \left\{ \sup\limits_{u \in (t + \frac{1}{2}, t+1]} S_u \leq S_a \right\}
$$

La PDD cumulée sur $T$ périodes est donnée par :

$$
\text{PDD}_T = \mathbb{E}^{\mathbb{Q}} \left[ \sum_{t=0}^{T-1} \lambda_t(r, \sigma, S_a, S_0) \right]
$$

---

### Franchissement de barrière et pont brownien

Entre deux pas de temps $t_1$ et $t_2$, on utilise la propriété du **pont brownien** pour calculer la probabilité de franchissement d’une barrière \(M\) :

$$
\mathbb{P} \left( \sup_{t \in [t_1, t_2]} Z_t > M \right) = \exp \left( - \frac{2(a - M)(b - M)}{t_2 - t_1} \right)
$$

où $a = Z_{t_1}$ et $b = Z_{t_2}$.  
Cette formule corrige le biais des schémas de discrétisation.

---


$$
\lambda_t(r, \sigma, S_a, S_0) = \left( S_a - S_{t+1} \right)^+ \cdot \mathbf{1}_{ \{ S_{t+1} \leq \alpha S_a \} } \cdot \mathbf{1}_{ \left\{ \sup_{u \in (t + \frac{1}{2}, t+1]} S_u \leq S_a \right\} }
$$

La Provision pour Dépréciation Durable (PDD) cumulée sur \(T\) périodes est donnée par :  

$$
\text{PDD}_T = \mathbb{E}^{\mathbb{Q}} \left[ \sum_{t=0}^{T-1} \lambda_t(r, \sigma, S_a, S_0) \right]
$$

---

### Dépendances entre actifs

Pour un portefeuille de $d$ actifs $S^1, \ldots, S^d$, on modélise les corrélations en simulant un vecteur gaussien multivarié via la décomposition de Cholesky :

$$
R_t = \mu_t + L Z_t
$$

où :
- $R_t = (\log(S^1_t/S^1_{t-1}), \ldots, \log(S^d_t/S^d_{t-1}))$,
- $\mu_t = \left( (r - \frac{\sigma_1^2}{2})t, \ldots, (r - \frac{\sigma_d^2}{2})t \right)$,
- $L$ est la racine de Cholesky de la matrice de covariance \(\Sigma\),
- $Z_t$ est un vecteur standard normal.

---

### Sensibilités (Greeks)

On estime les dérivées partielles par différences finies :  

- **Delta** (sensibilité à $S_0$) :

$$
\Delta = \frac{\partial \text{PDD}}{\partial S_0}
$$

- **Vega** (sensibilité à $\sigma$) :

$$
\text{Vega} = \frac{\partial \text{PDD}}{\partial \sigma}
$$

- **Rho** (sensibilité à $r$) :

$$
\text{Rho} = \frac{\partial \text{PDD}}{\partial r}
$$

Les sensibilités sont calculées selon la formule des différences finies centrées :

$$
\frac{\partial f}{\partial x} \approx \frac{f(x + h) - f(x - h)}{2h}
$$

---

## Méthodes de simulation

- **Méthode naïve** : schéma d’Euler ou solution exacte, vérification des conditions sur chaque pas de temps.
- **Pont brownien** : probabilité analytique de franchissement entre deux points, meilleure précision sans réduire \(\Delta t\).

---

## Extensions

- **Simulation multi-périodes** : recalcul des pertes chaque année.
- **Portefeuille multi-actifs** : corrélation des actifs simulée avec Cholesky.
- **Model point** : simplification du portefeuille en un actif unique.

---

## Utilisation

Le code Python est structuré en modules et notebooks :
- Simulation des trajectoires d’actifs.
- Calcul des PDD et détection des franchissements.
- Analyse des sensibilités et des scénarios.

---

## Licence

Projet open-source sous licence libre. Contributions bienvenues !

